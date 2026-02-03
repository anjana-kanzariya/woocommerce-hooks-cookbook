## Custom Checkout Fields - Full Lifecycle

## Use case
- Add a custom field to the checkout form  
- Add an additional delivery date field  
- Validate the values before order creation  
- Save them to order meta  
- Display them in admin  
- Modify existing WooCommerce checkout fields (label, priority, required)

## Data lifecycle
Checkout form → validation → order meta → admin display → optional user meta sync

## Hooks used
- `woocommerce_checkout_fields` - Add or modify checkout fields  
- `woocommerce_checkout_process` - Validate posted data  
- `woocommerce_checkout_update_order_meta` - Save data to order  
- `woocommerce_admin_order_data_after_billing_address` - Display in admin  
- `woocommerce_checkout_posted_data` - Pre-fill checkout data  
- `wp_ajax_*` - Optional AJAX update of user profile during checkout

---

## 1. Add Custom Checkout Fields

```php
add_filter('woocommerce_checkout_fields', function ($fields) {

    $fields['billing']['custom_reference'] = [
        'type'        => 'text',
        'label'       => 'Purchase Reference',
        'placeholder' => 'Enter reference',
        'required'    => true,
        'priority'    => 25,
        'class'       => ['form-row-wide'],
    ];

    $fields['billing']['delivery_date'] = [
        'type'        => 'date',
        'label'       => 'Delivery Date',
        'required'    => false,
        'priority'    => 26,
        'class'       => ['form-row-wide'],
    ];

    return $fields;
});
```

---

## 2. Validate Fields on Checkout

```php
add_action('woocommerce_checkout_process', function () {

    if (empty($_POST['custom_reference'])) {
        wc_add_notice('Please enter a purchase reference.', 'error');
    }

    if (!empty($_POST['custom_reference']) && strlen($_POST['custom_reference']) > 20) {
        wc_add_notice('Purchase reference must be max 20 characters.', 'error');
    }

    if (!empty($_POST['delivery_date'])) {
        $date = sanitize_text_field($_POST['delivery_date']);

        if (strtotime($date) < strtotime('today')) {
            wc_add_notice('Delivery date cannot be in the past.', 'error');
        }
    }

});
```

---

## 3. Save Fields to Order Meta

```php
add_action('woocommerce_checkout_update_order_meta', function ($order_id) {

    if (!empty($_POST['custom_reference'])) {
        update_post_meta(
            $order_id,
            'custom_reference',
            sanitize_text_field($_POST['custom_reference'])
        );
    }

    if (!empty($_POST['delivery_date'])) {
        update_post_meta(
            $order_id,
            'delivery_date',
            sanitize_text_field($_POST['delivery_date'])
        );
    }

});
```

---

## 4. Display Values in Admin Order Screen

### Show below billing address

```php
add_action('woocommerce_admin_order_data_after_billing_address', function ($order) {

    $reference = get_post_meta($order->get_id(), 'custom_reference', true);
    $delivery  = get_post_meta($order->get_id(), 'delivery_date', true);

    if ($reference) {
        echo '<p><strong>Reference:</strong> ' . esc_html($reference) . '</p>';
    }

    if ($delivery) {
        echo '<p><strong>Delivery Date:</strong> ' . esc_html($delivery) . '</p>';
    }

});
```

---

## 5. Alternative: Dedicated Meta Box in Order Page

```php
add_action('add_meta_boxes', function () {
    add_meta_box(
        'custom_checkout_meta',
        'Checkout Information',
        function ($post) {

            $reference = get_post_meta($post->ID, 'custom_reference', true);
            $delivery  = get_post_meta($post->ID, 'delivery_date', true);

            echo '<p><strong>Reference:</strong> ' . esc_html($reference) . '</p>';
            echo '<p><strong>Delivery Date:</strong> ' . esc_html($delivery) . '</p>';
        },
        'shop_order',
        'side'
    );
});
```

---

## 6. Modify Existing Checkout Fields

```php
add_filter('woocommerce_checkout_fields', function ($fields) {

    // Changes label of the checkout fields
    $fields['billing']['billing_email']['label'] = 'Order confirmation email';
    $fields['billing']['billing_phone']['label'] = 'SMS notification number';

    // Sets the priority. Helpful in setting the sequence of the fields.
    $fields['billing']['billing_first_name']['priority'] = 10;
    $fields['billing']['billing_last_name']['priority']  = 20;
    $fields['billing']['billing_email']['priority']      = 30;
    $fields['billing']['billing_phone']['priority']      = 40;

    // Make/ remove a field from being mandatory.
    $fields['billing']['billing_phone']['required'] = false;

    return $fields;
});
```

---

## 7. Remove Default Fields

```php
add_filter('woocommerce_checkout_fields', function ($fields) {

    unset($fields['billing']['billing_company']);
    unset($fields['billing']['billing_address_1']);
    unset($fields['billing']['billing_address_2']);
    unset($fields['billing']['billing_postcode']);
    unset($fields['billing']['billing_city']);

    return $fields;
});
```

---

## 8. Pre-Fill Checkout Data Programmatically

```php
add_filter('woocommerce_checkout_posted_data', function ($data) {

    if (!is_checkout()) {
        return $data;
    }

    $user_id = get_current_user_id();

    $data['billing_first_name'] = get_user_meta($user_id, 'billing_first_name', true);
    $data['billing_last_name']  = get_user_meta($user_id, 'billing_last_name', true);
    $data['billing_phone']      = get_user_meta($user_id, 'billing_phone', true);

    return $data;
});
```

---

## 9. Optional: AJAX Save During Checkout

```php
add_action('wp_ajax_nopriv_update_checkout_profile', 'update_checkout_profile');

function update_checkout_profile() {

    parse_str($_POST['form_data'], $form);

    $user_id = get_current_user_id();

    update_user_meta($user_id, 'billing_first_name', sanitize_text_field($form['billing_first_name']));
    update_user_meta($user_id, 'billing_last_name', sanitize_text_field($form['billing_last_name']));
    update_user_meta($user_id, 'billing_phone', sanitize_text_field($form['billing_phone']));

    wp_send_json_success();
}
```

---

## Notes

- Checkout fields must be added using `woocommerce_checkout_fields`, not plain HTML.  
- Validation should always be done in `woocommerce_checkout_process`.  
- Custom checkout data is **not saved automatically** — it must be written in `woocommerce_checkout_update_order_meta`.  
- Admin display requires manual reading from order meta.  
- Field priority controls visual order in the form.  
- Removing a field with `unset()` does not affect stored order data.  
- Use AJAX only for optional profile updates, not critical order data.

---

## Common Pitfalls

- Trusting client-side validation only  
- Forgetting to sanitize `$_POST` values  
- Expecting custom fields to auto-persist  
- Using user meta instead of order meta for per-order values  
- Changing field keys after orders already exist