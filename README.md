# WooCommerce Hooks Cookbook

A collection of real-world WooCommerce hooks examples with clear use cases, lifecycle, and best practices.

## About

This repository provides:

- Complete hook usage examples with context
- Cart item data lifecycle: POST → cart → order
- Validation patterns for cart and checkout
- Clear documentation for each snippet

## Text Domain & Copy-Paste Note

All example code in this repository uses `'woocommerce'` as the text domain for translation functions (e.g., `__()` and `_e()`).  
If you copy this code into your own plugin or theme, replace `'woocommerce'` with your own text domain to ensure proper translation handling.

Also, be cautious when copying hooks/code directly — adapt variable names, product IDs, and custom keys to match your project context.

## Current Examples

1. **Add Custom Cart Item Data**  
   Adds custom fields to cart items and persists them into the order.  
   File: `add-custom-cart-item-data.md`

2. **Validate Cart Items**  
   Validates cart items based on custom fields before checkout.  
   File: `validate-cart-items.md`

## Contribution

- Follow the same structure for new examples:  
  **Use case → Data lifecycle → Hooks → Code → Notes**
- Submit pull requests with one scenario per MD file.
