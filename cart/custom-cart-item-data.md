## Persist Custom Cart Item Data from Add to Cart to Order

## Use case
Add custom data (e.g. booking number, project code, comment, etc) to a cart item and persist it into the order.

## Data lifecycle
POST → cart item → order item → read from order

## Hooks used
- `woocommerce_add_cart_item_data` - Adds custom data to the cart item before it is stored.
- `woocommerce_get_item_data` - Displays custom cart item data in Cart & Checkout.
- `woocommerce_checkout_create_order_line_item` - Persists cart item data into the order line item.
- `woocommerce_checkout_order_processed` - Example hook showing how order item meta can be accessed later.

## Code
```php
add_filter('woocommerce_add_cart_item_data', function ($cart_item_data, $product_id) {
    
    if (!empty($_POST['custom_key'])) {
        $cart_item_data['custom_key'] = sanitize_text_field($_POST['custom_key']);
    }

    // Force unique cart item
    // This is helpful in cases where same product can have multiple custom key values and the cart items should not be merged.
    // $cart_item_data['unique_key'] = md5(microtime() . rand());

    return $cart_item_data;
}, 10, 2);

add_filter('woocommerce_get_item_data', function ($item_data, $cart_item) {

    if (!empty($cart_item['custom_key'])) {
        $item_data[] = [
            'name'  => __('Custom Key', 'woocommerce'),
            'value' => esc_html($cart_item['custom_key']),
        ];
    }

    return $item_data;
}, 10, 2);


add_action('woocommerce_checkout_create_order_line_item', function ($item, $cart_item_key, $values, $order) {
    if (!empty($values['custom_key'])) {
        // Third parameter ensures the meta key is unique per order item
        $item->add_meta_data('custom_key', $values['custom_key'], true);
    }
}, 10, 4);


add_action('woocommerce_checkout_order_processed', function ($order_id) {
    $order = wc_get_order($order_id);
    foreach ($order->get_items() as $item) {
        $custom_key = $item->get_meta('custom_key');

        if ($custom_key) {
            // Use $custom_key here
        }
    }
});
```

## Notes
- Cart item meta does not automatically persist to the order — it must be explicitly copied.
- `woocommerce_checkout_create_order_line_item` is mandatory to persist data in order.
- Order item meta is accessed via the order item object (`$item->get_meta()`), not from the order object directly.
