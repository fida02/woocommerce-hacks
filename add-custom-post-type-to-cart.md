# Add Custom Post Type to WooCommerce Cart
Suppose we have a custom post type named **"book"**. Now we want to add this post type to the WooCommerce cart. But WooCommerce doesn't allow that. WooCommerce only allows post type **"product"** to add. 

So, we need a little hack so that WooCommerce accept our custom post type item. 
```php
<?php

/**
 * Enable "book" post type to WooCommerce cart
 * 
 * Extend WooCommerce Product data
 *
 * WC Tested up to: 6.3.1
 */
class Fida_Product_Data_Store_CPT extends WC_Product_Data_Store_CPT implements WC_Object_Data_Store_Interface, WC_Product_Data_Store_Interface {


    /**
     * Method to read a product from the database.
     * @param WC_Product
     */
    public function read( &$product ) {
        $product->set_defaults();
        
        // Adding "book" post type in the condition to allow
        if ( (! $product->get_id() || ! ( $post_object = get_post( $product->get_id() ) ) || 'product' !== $post_object->post_type) && 'book' !== $post_object->post_type ) {
            throw new Exception( __( 'Invalid product.', 'woocommerce' ) );
        }

        $id = $product->get_id();

        $product->set_props( array(
            'name'              => $post_object->post_title,
            'slug'              => $post_object->post_name,
            'date_created'      => 0 < $post_object->post_date_gmt ? wc_string_to_timestamp( $post_object->post_date_gmt ) : null,
            'date_modified'     => 0 < $post_object->post_modified_gmt ? wc_string_to_timestamp( $post_object->post_modified_gmt ) : null,
            'status'            => $post_object->post_status,
            'description'       => $post_object->post_content,
            'short_description' => $post_object->post_excerpt,
            'parent_id'         => $post_object->post_parent,
            'menu_order'        => $post_object->menu_order,
            'reviews_allowed'   => 'open' === $post_object->comment_status,
        ) );

        $this->read_attributes( $product );
        $this->read_downloads( $product );
        $this->read_visibility( $product );
        $this->read_product_data( $product );
        $this->read_extra_data( $product );
        $product->set_object_read( true );
    }


}

function fida_woocommerce_data_stores( $stores ) {

  // This file path may change on future WooCommerce release
  require_once WP_PLUGIN_DIR . '/woocommerce/includes/class-wc-data-store.php';
  $stores['product'] = 'Fida_Product_Data_Store_CPT';

  return $stores;
}
add_filter( 'woocommerce_data_stores', 'fida_woocommerce_data_stores' );

/**
 * Get price from custom meta field and set as WooCommerce price
 *
 * WooCommerce requires price to add items to cart
 * "price" custom meta field is created in the "book" post type
 */
function fida_woocommerce_product_get_price( $price, $product ) {

    $post_id = $product->get_id();

    if (get_post_type($post_id) === 'book'){
        $price = get_post_meta($post_id, "price", true);
    }
    return $price;
}
add_filter('woocommerce_product_get_price', 'fida_woocommerce_product_get_price', 10, 2 );
?>
 ```
