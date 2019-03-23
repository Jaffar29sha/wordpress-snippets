# wordpress-snippets
Wordpress and woocommerce shortcodes

/* HIDE OTHER SHIPPING METHODS IF FREE SHIPPING AVAILABLE */

add_filter( 'woocommerce_package_rates', 'my_hide_shipping_when_free_is_available', 100 );
function my_hide_shipping_when_free_is_available( $rates ) {
	$free = array();
	foreach ( $rates as $rate_id => $rate ) {
		if ( 'free_shipping' === $rate->method_id ) {
			$free[ $rate_id ] = $rate;
			break;
		}
	}
	return ! empty( $free ) ? $free : $rates;
}

/* CHANGE ADD TO CART TEXT IN SINGLE PRODUCT PAGE */

add_filter('woocommerce_product_single_add_to_cart_text', 'woo_custom_cart_button_text');  
function woo_custom_cart_button_text() {
    return __('Add Item', 'woocommerce');
}

/* CHANGE ADD TO CART TEXT IN PRODUCT LIST/GRID PAGE */

add_filter( 'woocommerce_product_add_to_cart_text', 'woo_custom_product_add_to_cart_text');
function woo_custom_product_add_to_cart_text() {
	return __( 'Add Item', 'woocommerce' );
}

/* ADD QUANTITY BUTTON PRODUCT LIST/GRID PAGE */

function ace_shop_page_add_quantity_field() {
	$product = wc_get_product( get_the_ID() );
	if ( ! $product->is_sold_individually() && 'variable' != $product->get_type() && $product->is_purchasable() ) {
		woocommerce_quantity_input( array( 'min_value' => 0, 'max_value' => $product->backorders_allowed() ? '' : $product->get_stock_quantity() ) );
	}
}
add_action( 'woocommerce_after_shop_loop_item', 'ace_shop_page_add_quantity_field', 12 );

function ace_shop_page_quantity_add_to_cart_handler() {
	wc_enqueue_js( '
		$(".woocommerce .products").on("click", ".quantity input", function() {
			return false;
		});
		$(".woocommerce .products").on("change input", ".quantity .qty", function() {
			var add_to_cart_button = $(this).parents( ".product" ).find(".add_to_cart_button");
			// For AJAX add-to-cart actions
			add_to_cart_button.data("quantity", $(this).val());
			// For non-AJAX add-to-cart actions
			add_to_cart_button.attr("href", "?add-to-cart=" + add_to_cart_button.attr("data-product_id") + "&quantity=" + $(this).val());
		});
		// Trigger on Enter press
		$(".woocommerce .products").on("keypress", ".quantity .qty", function(e) {
			if ((e.which||e.keyCode) === 13) {
				$( this ).parents(".product").find(".add_to_cart_button").trigger("click");
			}
		});
	' );
}
add_action( 'init', 'ace_shop_page_quantity_add_to_cart_handler' );
