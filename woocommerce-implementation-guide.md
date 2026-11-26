# Healing Hastha — WooCommerce Technical Implementation Guide
**How to Add Apollo, MedPlus & Amazon Multi-Channel Buttons to WooCommerce**

This guide provides the exact WordPress & WooCommerce code to implement the multi-channel retail availability directly on `healinghastha.com`.

---

## Method 1: Using Custom Fields in WooCommerce Products (Recommended)

### Step 1: Add Custom Product Fields for Retail Links
Add this snippet into your child theme's `functions.php` or using the **Code Snippets** plugin. This adds simple URL inputs in the WooCommerce product edit screen:

```php
<?php
// 1. Add Retailer URLs to WooCommerce Product Data Tab (General tab)
add_action('woocommerce_product_options_general_product_data', 'hh_add_retailer_custom_fields');
function hh_add_retailer_custom_fields() {
    echo '<div class="options_group" style="border-top: 1px solid #eee; padding-top: 10px;">';
    echo '<p><strong>' . __('Multi-Channel Retail Availability Links', 'woocommerce') . '</strong></p>';
    
    woocommerce_wp_text_input(array(
        'id'          => '_hh_apollo_url',
        'label'       => __('Apollo Pharmacy URL', 'woocommerce'),
        'placeholder' => 'https://www.apollopharmacy.in/...',
        'desc_tip'    => 'true',
        'description' => __('Enter the product page URL on Apollo Pharmacy.', 'woocommerce')
    ));

    woocommerce_wp_text_input(array(
        'id'          => '_hh_medplus_url',
        'label'       => __('MedPlus URL', 'woocommerce'),
        'placeholder' => 'https://www.medplusmart.com/...',
        'desc_tip'    => 'true',
        'description' => __('Enter the product page URL on MedPlus.', 'woocommerce')
    ));

    woocommerce_wp_text_input(array(
        'id'          => '_hh_amazon_url',
        'label'       => __('Amazon India URL', 'woocommerce'),
        'placeholder' => 'https://www.amazon.in/dp/...',
        'desc_tip'    => 'true',
        'description' => __('Enter the Amazon product listing URL.', 'woocommerce')
    ));
    echo '</div>';
}

// 2. Save Custom Retailer URLs when Product is Updated
add_action('woocommerce_process_product_meta', 'hh_save_retailer_custom_fields');
function hh_save_retailer_custom_fields($post_id) {
    if (isset($_POST['_hh_apollo_url'])) {
        update_post_meta($post_id, '_hh_apollo_url', esc_url_raw($_POST['_hh_apollo_url']));
    }
    if (isset($_POST['_hh_medplus_url'])) {
        update_post_meta($post_id, '_hh_medplus_url', esc_url_raw($_POST['_hh_medplus_url']));
    }
    if (isset($_POST['_hh_amazon_url'])) {
        update_post_meta($post_id, '_hh_amazon_url', esc_url_raw($_POST['_hh_amazon_url']));
    }
}
```

---

## Step 2: Render Retailer Badges & Buttons in Shop / Product Cards

Hook into `woocommerce_after_shop_loop_item` to display the "Also available at" strip below the primary Add to Cart button:

```php
<?php
add_action('woocommerce_after_shop_loop_item', 'hh_render_retailer_buttons_in_loop', 15);
function hh_render_retailer_buttons_in_loop() {
    global $product;
    $product_id = $product->get_id();

    $apollo_url  = get_post_meta($product_id, '_hh_apollo_url', true);
    $medplus_url = get_post_meta($product_id, '_hh_medplus_url', true);
    $amazon_url  = get_post_meta($product_id, '_hh_amazon_url', true);

    // If product has at least one external retailer
    if ($apollo_url || $medplus_url || $amazon_url) {
        echo '<div class="hh-retailer-strip" style="margin-top: 10px; padding: 8px; background: #fafaf9; border-radius: 6px; border: 1px solid #e7e5e4;">';
        echo '<div style="font-size: 11px; font-weight: 700; color: #57534e; text-transform: uppercase; margin-bottom: 6px;">Also Available At:</div>';
        echo '<div style="display: flex; flex-wrap: wrap; gap: 6px;">';

        if ($apollo_url) {
            echo '<a href="' . esc_url($apollo_url) . '" target="_blank" rel="noopener" class="hh-chip-btn apollo">Apollo</a>';
        }
        if ($medplus_url) {
            echo '<a href="' . esc_url($medplus_url) . '" target="_blank" rel="noopener" class="hh-chip-btn medplus">MedPlus</a>';
        }
        if ($amazon_url) {
            echo '<a href="' . esc_url($amazon_url) . '" target="_blank" rel="noopener" class="hh-chip-btn amazon">Amazon</a>';
        }

        echo '</div>';
        echo '</div>';
    }
}
```

---

## Step 3: Add Accompanying CSS into WordPress Customizer

Paste into **Appearance > Customize > Additional CSS**:

```css
/* Multi-Channel Retailer Chips */
.hh-chip-btn {
  flex: 1;
  min-width: 65px;
  padding: 5px 8px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 700;
  text-decoration: none;
  text-align: center;
  display: inline-block;
  transition: all 0.2s ease;
}

.hh-chip-btn.apollo {
  background: #e6f6f4;
  color: #027668;
  border: 1px solid #99f6e4;
}
.hh-chip-btn.apollo:hover {
  background: #027668;
  color: #ffffff;
}

.hh-chip-btn.medplus {
  background: #fde8e8;
  color: #e53935;
  border: 1px solid #fecaca;
}
.hh-chip-btn.medplus:hover {
  background: #e53935;
  color: #ffffff;
}

.hh-chip-btn.amazon {
  background: #fffbeb;
  color: #92400e;
  border: 1px solid #fde68a;
}
.hh-chip-btn.amazon:hover {
  background: #111827;
  color: #fbbf24;
}
```

---

## Step 4: Product Configuration Checklist

1. **Sleep 8:**
   - Apollo Pharmacy URL: `https://www.apollopharmacy.in/search-medicines/Sleep%208`
   - MedPlus URL: `https://www.medplusmart.com/search?q=Healing+Hastha+Sleep+8`
2. **Relx Oil:**
   - MedPlus URL: `https://www.medplusmart.com/search?q=Healing+Hastha+Relx+Oil`
   - Apollo Pharmacy URL: `https://www.apollopharmacy.in/search-medicines/Relx%20Oil`
   - Amazon URL: `https://www.amazon.in/s?k=healing+hastha+relx+oil`
3. **Dr. MAYA Pain Relief Oil:**
   - Amazon URL: `https://www.amazon.in/s?k=healing+hastha+dr+maya+pain+relief+oil`
4. **All other products:**
   - Leave external retailer fields empty — they will automatically show as **Direct Exclusive Formulations**!
