## Various scenarios to validate cart items based on the custom key.

## Use case
Validation for custom cart item data such as:
- Personalized message for a customized greeting card
- Booking number or reservation ID
- Project or reference code tied to the product

## Data lifecycle
cart item → validation → block checkout (error notice)

## Hooks used
`woocommerce_check_cart_items` - Runs on cart & checkout load and is the primary validation hook

## Code
```php

// Check for mandatory custom key
add_action('woocommerce_check_cart_items', function () {

    if (!WC()->cart || WC()->cart->is_empty()) {
        return;
    }

    foreach (WC()->cart->get_cart() as $cart_item) {

        if (empty($cart_item['custom_key'])) {
            wc_add_notice(
                __('One or more products are missing custom key.', 'woocommerce'),
                'error'
            );
            return;
        }
    }
});


// Checks for data for a custom key. Can be extended to validate other cart item data in a similar manner
add_action('woocommerce_check_cart_items', function () {

    if (!WC()->cart || WC()->cart->is_empty()) {
        return;
    }

    foreach (WC()->cart->get_cart() as $cart_item) {

        // Checks if the custom key is numeric. Any other type of check or regex comparison can be done here
        if (!empty($cart_item['custom_key']) && !ctype_digit($cart_item['custom_key'])) {
            wc_add_notice(
                __('Invalid Custom Key detected. Please check your input.', 'woocommerce'),
                'error'
            );
            return;
        }
    }
});

// Prevent duplicate custom_key values across cart items. Useful for keys like booking number, reference IDs, etc.
add_action('woocommerce_check_cart_items', function () {

    if (!WC()->cart || WC()->cart->is_empty()) {
        return;
    }

    $keys = [];

    foreach (WC()->cart->get_cart() as $cart_item) {

        if (empty($cart_item['custom_key'])) {
            continue;
        }

        if (in_array($cart_item['custom_key'], $keys, true)) {
            wc_add_notice(
                __('Duplicate Custom Key found in cart items.', 'woocommerce'),
                'error'
            );
            return;
        }

        $keys[] = $cart_item['custom_key'];
    }
});

// Validate only for a specific product (or product IDs)
add_action('woocommerce_check_cart_items', function () {

    if (!WC()->cart || WC()->cart->is_empty()) {
        return;
    }

    // Product IDs can also be fetched from custom fields, options.
    $restricted_product_ids = [123, 456];

    foreach (WC()->cart->get_cart() as $cart_item) {

        if (!in_array($cart_item['product_id'], $restricted_product_ids, true)) {
            continue;
        }

        if (empty($cart_item['custom_key'])) {
            wc_add_notice(
                __('Custom Key is required for selected product.', 'woocommerce'),
                'error'
            );
            return;
        }
    }
});

// Same as required validation but checks only on checkout page.
add_action('woocommerce_check_cart_items', function () {
    if (!is_checkout() || !WC()->cart || WC()->cart->is_empty()) {
        return;
    }

    foreach (WC()->cart->get_cart() as $cart_item) {

        if (empty($cart_item['custom_key'])) {
            wc_add_notice(
                __('Please fill all required fields before checkout.', 'woocommerce'),
                'error'
            );
            return;
        }
    }
});



```

## Notes
- Each scenario is intentionally separated into its own hook for clarity and reuse. In production, these can be merged if needed.
- Keep validation logic lightweight — this hook runs on every cart and checkout load.
- woocommerce_check_cart_items is the correct hook for cart/checkout validation
- Validation errors must use `wc_add_notice( ..., 'error' )` to display WooCommerce blocking notices.
- Returning early prevents duplicate notices
- Cart validation runs before order creation
- Never validate order-level data here — cart only
