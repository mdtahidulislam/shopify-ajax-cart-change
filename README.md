# AJAX Cart API: change.js

## Key points:

## Steps

### Update cart when quantity is updated
***-- create quantity-input element***
```liquid
<quantity-input>
	<input
		id="quantity-{{ forloop.index }}"
		data-index="{{ forloop.index }}"
		type="number"
		name="updates[]"
		value="{{ item.quantity }}"
	>
</quantity-input>
```
***-- add onChange event & check: cart.js***
```js
class CartItems extends HTMLElement {
	constructor() {
		super()
		this.addEventListener('change', this.handleChange.bind(this))
	}

	handleChange(e) {
		console.log(e)
	}
}
```
***-- create fetch API: updateQuantity()***
```js
updateQuantity(line, quantity, sections) {
	console.log(line, quantity, sections)

	const data = {
		line,
		quantity,
		sections
	}

	fetch(`${window.Shopify.routes.root}cart/change.js`, {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
			'X-Requested-With': 'XMLHttpRequest'
		},
		body: JSON.stringify(data)
	})
		.then(response => {
			console.log(response)
			return response.json()
		})
		.then(data => {
			console.log(data)
		})
		.catch(error => {
			console.error('AJAX Cart Error:', error)
		})
}
```
***-- update handleChange()***
```js
...
const index = e.target.dataset.index
const quantity = e.target.value
const sections = 'main-cart-items,cart-icon-bubble,cart-drawer'
this.updateQuantity(index, quantity, sections)
***-- dispatch cart:change event***
```js
if (data.sections) {
	document.dispatchEvent(
		new CustomEvent('cart:change', {
			detail: {
				sections: data.sections
			}
		})
	)
}
```

***-- listen the event & refactor global.js***
```js
document.addEventListener('cart:updated', e => {
	const sections = e.detail.sections
	bundledSections(sections)
	openCartDrawer()
})

document.addEventListener('cart:change', e => {
	const sections = e.detail.sections
	bundledSections(sections)
})

function bundledSections(sections) {
	Object.keys(sections).forEach(sectionId => {
		const target = document.getElementById(sectionId) || document.getElementById(`shopify-section-${sectionId}`)
		if (target) {
			console.log(`Updating section: ${sectionId}`)
			const parsedHTML = new DOMParser().parseFromString(sections[sectionId], 'text/html')
			const parsedContent = parsedHTML.querySelector('.shopify-section')
			if (parsedContent) {
				target.innerHTML = content.innerHTML
			} else {
				// Fallback if shopify-section wrapper isn't found
				target.innerHTML = sections[sectionId]
			}
		} else {
			console.warn(`Target not found for section: ${sectionId}`)
		}
	})
}
```




### update cart remove button: main-cart-items.liquid
***--add cart-remove-button***
```liquid
<cart-remove-button
  id="cart-remove-{{ forloop.index }}"
  data-index="{{ forloop.index }}"
>
  <a href="{{ item.url_to_remove }}">REMOVE</a>
</cart-remove-button>
```
***--create cartremovebutton component & event listener***
```js
class CartRemoveButton extends HTMLElement {
	connectedCallback() {
		this.addEventListener('click', this.removeCartItem.bind(this))
	}

	disconnectedCallback() {
		this.removeEventListener('click', this.removeCartItem.bind(this))
	}

	removeCartItem(e) {
		e.preventDefault()
		console.log(e)
	}
}

if (!customElements.get('cart-remove-button')) {
	customElements.define('cart-remove-button', CartRemoveButton)
```

***--add script & check***
```html
<script src="{{ 'product-form.js' | asset_url }}" defer></script>
```

***-- create data object**
```js
const data = {
  line: this.dataset.index,
  quantity: 0
}
```

***-- create fetch API & log response**
```js
fetch(`${window.Shopify.routes.root}cart/change.js`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
})
  .then(response => {
    console.log(response)
    return response.json()
  })
  .then(data => {
    console.log(data)
  })
  .catch(error => {
    console.error('AJAX Cart Error:', error)
  })
```
***-- add bundled section rendering***
```js
const data = {
  ...
  sections: 'main-cart-items,cart-icon-bubble,cart-drawer'
}
```
***-- dispatch new event(cart:removed)***
```js
if (data.sections) {
  document.dispatchEvent(
    new CustomEvent('cart:removed', {
      detail: {
        sections: data.sections
      }
    })
  )
}
```

***-- prevent cart-drawer from closing/flickering***
```js
...
// Special handling for cart drawer to prevent it from closing/flickering
if (sectionId === 'cart-drawer') {
  const existingDrawer = document.getElementById('cartDrawer')
  const newDrawerBody = parsedHTML.querySelector('.offcanvas-body')

  if (existingDrawer && newDrawerBody) {
    const existingBody = existingDrawer.querySelector('.offcanvas-body')
    if (existingBody) {
      existingBody.innerHTML = newDrawerBody.innerHTML
      return
    }
  }
}

const parsedContent = parsedHTML.querySelector('.shopify-section')

if (parsedContent) {
...
```
***-- ***
