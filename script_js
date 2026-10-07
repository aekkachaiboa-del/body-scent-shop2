/* =========================================================
   Body Scent — Shared JavaScript
   ใช้ไฟล์เดียวร่วมกันทุกหน้า
   ========================================================= */

document.addEventListener("DOMContentLoaded", () => {
  if (document.querySelector("#product-list")) {
    initProductPage();
  }

  if (document.querySelector("#orderForm")) {
    initOrderPage();
  }

  if (document.querySelector("#ordersTable")) {
    initAdminPage();
  }
});

/* =========================================================
   Utilities
   ========================================================= */

function getQueryParams() {
  return new URLSearchParams(window.location.search);
}

function formatPrice(price) {
  const number = Number(price);
  return Number.isFinite(number) ? `${number.toLocaleString("th-TH")} บาท` : "0 บาท";
}

function normalizeMood(value) {
  const allowed = ["fresh", "sweet", "confident", "romance"];
  const mood = String(value || "").trim().toLowerCase();
  return allowed.includes(mood) ? mood : "all";
}

function safeText(value) {
  return value == null ? "" : String(value);
}

/* =========================================================
   product.html
   ========================================================= */

async function initProductPage() {
  const productList = document.querySelector("#product-list");
  const filterBar = document.querySelector("#filter-bar");

  let products = [];
  let activeMood = normalizeMood(getQueryParams().get("mood"));

  try {
    const response = await fetch("products.json");

    if (!response.ok) {
      throw new Error(`โหลด products.json ไม่สำเร็จ (${response.status})`);
    }

    products = await response.json();

    if (!Array.isArray(products)) {
      throw new Error("รูปแบบข้อมูล products.json ไม่ถูกต้อง");
    }

    renderMoodFilters(filterBar, activeMood, (mood) => {
      activeMood = mood;
      renderProducts(productList, products, activeMood);
      updateMoodQueryParam(activeMood);
      updateActiveFilterButton(filterBar, activeMood);
    });

    renderProducts(productList, products, activeMood);
  } catch (error) {
    console.error("Body Scent product load error:", error);

    productList.replaceChildren();

    const message = document.createElement("p");
    message.className = "text-muted";
    message.textContent = "ไม่สามารถโหลดข้อมูลสินค้าได้ กรุณาลองใหม่อีกครั้ง";
    productList.appendChild(message);
  }
}

function renderMoodFilters(filterBar, activeMood, onFilter) {
  if (!filterBar) return;

  const filters = [
    { value: "all", label: "ทั้งหมด" },
    { value: "fresh", label: "Fresh" },
    { value: "sweet", label: "Sweet" },
    { value: "confident", label: "Confident" },
    { value: "romance", label: "Romance" }
  ];

  filterBar.replaceChildren();

  filters.forEach((filter) => {
    const button = document.createElement("button");
    button.type = "button";
    button.className = "btn btn-ghost filter-btn";
    button.dataset.mood = filter.value;
    button.textContent = filter.label;

    if (filter.value === activeMood) {
      button.classList.add("active");
      button.setAttribute("aria-pressed", "true");
    } else {
      button.setAttribute("aria-pressed", "false");
    }

    button.addEventListener("click", () => {
      onFilter(filter.value);
    });

    filterBar.appendChild(button);
  });
}

function updateActiveFilterButton(filterBar, activeMood) {
  if (!filterBar) return;

  filterBar.querySelectorAll("[data-mood]").forEach((button) => {
    const isActive = button.dataset.mood === activeMood;
    button.classList.toggle("active", isActive);
    button.setAttribute("aria-pressed", String(isActive));
  });
}

function updateMoodQueryParam(mood) {
  const url = new URL(window.location.href);

  if (mood === "all") {
    url.searchParams.delete("mood");
  } else {
    url.searchParams.set("mood", mood);
  }

  window.history.replaceState({}, "", url);
}

function renderProducts(container, products, mood = "all") {
  const filteredProducts =
    mood === "all"
      ? products
      : products.filter((product) => product.mood === mood);

  container.replaceChildren();

  if (filteredProducts.length === 0) {
    const empty = document.createElement("p");
    empty.className = "text-muted";
    empty.textContent = "ไม่พบสินค้าในหมวดนี้";
    container.appendChild(empty);
    return;
  }

  const fragment = document.createDocumentFragment();

  filteredProducts.forEach((product) => {
    fragment.appendChild(createProductCard(product));
  });

  container.appendChild(fragment);
}

function createProductCard(product) {
  const article = document.createElement("article");
  article.className = "card product-card";

  const imageWrap = document.createElement("div");
  imageWrap.className = "product-card__image";

  const image = document.createElement("img");
  image.src = safeText(product.image);
  image.alt = safeText(product.name);
  image.loading = "lazy";

  imageWrap.appendChild(image);

  const body = document.createElement("div");
  body.className = "product-card__body";

  const mood = document.createElement("span");
  mood.className = `mood-label mood-${safeText(product.mood)}`;
  mood.textContent = safeText(product.mood);

  const title = document.createElement("h3");
  title.className = "product-card__name";
  title.textContent = safeText(product.name);

  const meta = document.createElement("p");
  meta.className = "product-card__meta";
  meta.textContent = `${safeText(product.type)} • ${safeText(product.size)}`;

  const description = document.createElement("p");
  description.className = "text-muted";
  description.textContent = safeText(product.description);

  const price = document.createElement("div");
  price.className = "product-card__price";
  price.textContent = formatPrice(product.price);

  const orderLink = document.createElement("a");
  orderLink.className = "btn btn-accent";
  orderLink.textContent = "สั่งซื้อ";

  const params = new URLSearchParams({
    item: safeText(product.name),
    price: String(Number(product.price) || 0)
  });

  orderLink.href = `order.html?${params.toString()}`;

  body.append(mood, title, meta, description, price, orderLink);
  article.append(imageWrap, body);

  return article;
}

/* =========================================================
   order.html
   ========================================================= */

function initOrderPage() {
  const orderForm = document.querySelector("#orderForm");
  const itemsField = document.querySelector("#items");
  const totalField = document.querySelector("#total");

  const params = getQueryParams();

  const item = safeText(params.get("item")).trim();
  const parsedPrice = Number(params.get("price"));
  const price = Number.isFinite(parsedPrice) && parsedPrice >= 0 ? parsedPrice : 0;

  // ต้องเติมทั้งสองช่องเสมอ
  setFormFieldValue(itemsField, item);
  setFormFieldValue(totalField, price);

  orderForm.addEventListener("submit", (event) => {
    event.preventDefault();

    const customerNameField = document.querySelector("#customerName");
    const contactField = document.querySelector("#contact");
    const noteField = document.querySelector("#note");

    const payload = {
      customerName: getFormFieldValue(customerNameField),
      contact: getFormFieldValue(contactField),
      items: getFormFieldValue(itemsField),
      total: normalizeStoredTotal(getFormFieldValue(totalField)),
      note: getFormFieldValue(noteField),
      timestamp: new Date().toISOString()
    };

    try {
      const storageKey = "bodyScentOrders";
      const existingRaw = localStorage.getItem(storageKey);

      let orders = [];

      if (existingRaw) {
        const parsed = JSON.parse(existingRaw);
        orders = Array.isArray(parsed) ? parsed : [];
      }

      orders.push(payload);
      localStorage.setItem(storageKey, JSON.stringify(orders));

      window.location.href = "thankyou.html";
    } catch (error) {
      console.error("Body Scent order save error:", error);
      alert("ไม่สามารถบันทึกคำสั่งซื้อได้ กรุณาลองใหม่อีกครั้ง");
    }
  });
}

function setFormFieldValue(element, value) {
  if (!element) return;

  if (
    element instanceof HTMLInputElement ||
    element instanceof HTMLTextAreaElement ||
    element instanceof HTMLSelectElement
  ) {
    element.value = safeText(value);
  } else {
    element.textContent = safeText(value);
  }
}

function getFormFieldValue(element) {
  if (!element) return "";

  if (
    element instanceof HTMLInputElement ||
    element instanceof HTMLTextAreaElement ||
    element instanceof HTMLSelectElement
  ) {
    return safeText(element.value).trim();
  }

  return safeText(element.textContent).trim();
}

function normalizeStoredTotal(value) {
  const numeric = Number(String(value).replace(/[^\d.-]/g, ""));
  return Number.isFinite(numeric) ? numeric : 0;
}

/* =========================================================
   admin.html
   ========================================================= */

function initAdminPage() {
  const table = document.querySelector("#ordersTable");
  const tbody = table.querySelector("tbody");

  if (!tbody) return;

  tbody.replaceChildren();

  let orders = [];

  try {
    const raw = localStorage.getItem("bodyScentOrders");

    if (raw) {
      const parsed = JSON.parse(raw);
      orders = Array.isArray(parsed) ? parsed : [];
    }
  } catch (error) {
    console.error("Body Scent admin load error:", error);
    orders = [];
  }

  orders.sort((a, b) => {
    const timeA = new Date(a?.timestamp || 0).getTime();
    const timeB = new Date(b?.timestamp || 0).getTime();
    return timeB - timeA;
  });

  if (orders.length === 0) {
    renderEmptyOrdersRow(tbody, table);
    return;
  }

  const fragment = document.createDocumentFragment();

  orders.forEach((order) => {
    const row = document.createElement("tr");

    appendCell(row, formatThaiDate(order.timestamp));
    appendCell(row, order.customerName);
    appendCell(row, order.contact);
    appendCell(row, order.items);
    appendCell(row, formatPrice(order.total));
    appendCell(row, order.note || "-");

    fragment.appendChild(row);
  });

  tbody.appendChild(fragment);
}

function appendCell(row, value) {
  const cell = document.createElement("td");
  cell.textContent = safeText(value);
  row.appendChild(cell);
}

function formatThaiDate(timestamp) {
  if (!timestamp) return "-";

  const date = new Date(timestamp);

  if (Number.isNaN(date.getTime())) {
    return "-";
  }

  return date.toLocaleString("th-TH");
}

function renderEmptyOrdersRow(tbody, table) {
  const row = document.createElement("tr");
  const cell = document.createElement("td");

  const headerCount =
    table.querySelectorAll("thead th").length ||
    table.querySelectorAll("tr:first-child th").length ||
    1;

  cell.colSpan = headerCount;
  cell.textContent = "ยังไม่มีคำสั่งซื้อ";
  cell.style.textAlign = "center";

  row.appendChild(cell);
  tbody.appendChild(row);
}
