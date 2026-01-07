# Video Tutorial: Implementing AJAX Quantity Change in Shopify

This guide is designed for a technical coding content creator to produce a step-by-step video for beginners on how to implement AJAX-based quantity updates in a Shopify theme.

---

## **Video Title Idea**
"Level Up Your Shopify Theme: Smooth AJAX Quantity Updates (No Refresh!)"

## **Video Intro (0:00 - 0:45)**
*   **Hook:** "Stop letting your customers wait for page reloads! Today, we're building a sleek AJAX quantity change feature for your Shopify cart."
*   **Demo:** Show the cart page. Change a quantity, and watch the subtotal and total update instantly without the page flickering or reloading.
*   **Overview:** "We'll use Shopify's **Section Rendering API**, **Custom Elements**, and simple JavaScript."

---

## **Step 1: The Liquid HTML Structure (0:45 - 2:30)**

First, we need to wrap our cart items in a custom HTML element so we can easily listen for changes.

**File:** `sections/main-cart-items.liquid`

### **The Wrap**
Wrap your entire cart form or items list in a `<cart-items>` tag.
```liquid
<cart-items class="cart-items">
  <form action="{{ routes.cart_url }}" method="post">
    <!-- Cart Table Here -->
  </form>
</cart-items>
```

### **The Quantity Input**
Make sure your input has a `data-index` so we know which line item to update.
```liquid
<input
  id="quantity-{{ forloop.index }}"
  data-index="{{ forloop.index }}"
  type="number"
  name="updates[]"
  value="{{ item.quantity }}"
>
```

### **The Remove Button**
We can also create a custom element for the remove button to keep things modular.
```liquid
<cart-remove-button data-index="{{ forloop.index }}">
  <a href="{{ item.url_to_remove }}">REMOVE</a>
</cart-remove-button>
```

---

## **Step 2: The JavaScript Logic (2:30 - 5:00)**

Now, let's create the logic that sends data to Shopify without reloading the page.

**File:** `assets/cart.js`

### **1. Define the Custom Element**
```javascript
class CartItems extends HTMLElement {
  constructor() {
    super();
    // Listen for any 'change' event inside this element
    this.addEventListener('change', this.handleChange.bind(this));
  }

  handleChange(e) {
    const index = e.target.dataset.index;
    const quantity = e.target.value;
    const sections = 'main-cart-items,cart-icon-bubble,cart-drawer';
    this.updateQuantity(index, quantity, sections);
  }
}
```

### **2. The AJAX Fetch Request**
The magic happens here using Shopify's `/cart/change.js` endpoint.
```javascript
updateQuantity(line, quantity, sections) {
  const data = {
    line,
    quantity,
    sections // Request specific sections to be re-rendered
  };

  fetch(`${window.Shopify.routes.root}cart/change.js`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  })
  .then(response => response.json())
  .then(data => {
    // Dispatch a custom event with the new section HTML
    document.dispatchEvent(new CustomEvent('cart:change', {
      detail: { sections: data.sections }
    }));
  });
}
```

---

## **Step 3: Updating the UI (5:00 - 7:30)**

Once we get the fresh HTML back from Shopify, we need to swap it into the page.

**File:** `assets/global.js`

### **The Event Listener**
```javascript
document.addEventListener('cart:change', e => {
  const sections = e.detail.sections;
  
  // Loop through each returned section
  Object.keys(sections).forEach(sectionId => {
    const target = document.getElementById(sectionId) || document.getElementById(`shopify-section-${sectionId}`);
    
    if (target) {
      // Parse the HTML and update the target element
      const parsedHTML = new DOMParser().parseFromString(sections[sectionId], 'text/html');
      const content = parsedHTML.querySelector('.shopify-section') || parsedHTML.body;
      target.innerHTML = content.innerHTML;
    }
  });
});
```

---

## **Bonus: The Remove Button (7:30 - 8:30)**
Explain how simple it is to reuse the logic.
```javascript
class CartRemoveButton extends HTMLElement {
  constructor() {
    super();
    this.addEventListener('click', e => {
      e.preventDefault();
      const cartItems = this.closest('cart-items');
      // Setting quantity to 0 removes the item
      cartItems.updateQuantity(this.dataset.index, 0, 'main-cart-items');
    });
  }
}
```

## **Step 4: Bonus - Plus/Minus Buttons (8:30 - 9:30)**

To make it even better, let's add a custom element for the quantity input itself to handle plus and minus buttons.

**Liquid:**
```liquid
<quantity-input class="quantity-input">
  <button type="button" name="minus">-</button>
  <input
    id="quantity-{{ forloop.index }}"
    data-index="{{ forloop.index }}"
    type="number"
    name="updates[]"
    value="{{ item.quantity }}"
  >
  <button type="button" name="plus">+</button>
</quantity-input>
```

**JavaScript (assets/cart.js):**
```javascript
class QuantityInput extends HTMLElement {
  constructor() {
    super();
    this.input = this.querySelector('input');
    this.changeEvent = new Event('change', { bubbles: true });

    this.querySelectorAll('button').forEach(button =>
      button.addEventListener('click', this.onButtonClick.bind(this))
    );
  }

  onButtonClick(event) {
    event.preventDefault();
    const previousValue = this.input.value;

    if (event.currentTarget.name === 'plus') {
      this.input.stepUp();
    } else {
      this.input.stepDown();
    }

    if (previousValue !== this.input.value) {
      this.input.dispatchEvent(this.changeEvent);
    }
  }
}

customElements.define('quantity-input', QuantityInput);
```

---

## **Conclusion & Outro (9:30 - 10:00)**
*   **Summary:** "By using Custom Elements and the Section Rendering API, we've created a modular, fast, and modern cart experience."
*   **Call to Action:** "If you found this helpful, smash that like button and subscribe for more Shopify development tutorials!"

---

## **Tips for the Creator**
1.  **Visuals:** Use a split-screen view: Code on the left, Browser (Network Tab) on the right to show the fetch requests real-time.
2.  **Emphasis:** Highlight the `sections` parameter in the fetch request. This is the key to getting Shopify to return HTML instead of just JSON.
3.  **Error Handling:** Briefly mention adding a `loading` state to prevent users from clicking multiple times while a request is pending.
