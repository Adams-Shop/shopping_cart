[![Overall](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)
[![Security](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0%2Fsecurity)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)
[![Reliability](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0%2Freliability)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)
[![Scalability](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0%2Fscalability)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)
[![Observability](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0%2Fobservability)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)
[![Quality](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fapp.opslevel.com%2Fapi%2Fservice_level%2FfP8t9AVhq_fIq6zzKz-24ett7aidNb10I5lkyj3wRl0%2Fquality)](https://app.opslevel.com/services/shopping_cart_service/maturity-report)

# Shopping Cart

Usage
To get started, add the Buyable interface to your model.

use Illuminate\Database\Eloquent\Model;
use Treestoneit\ShoppingCart\Buyable;
use Treestoneit\ShoppingCart\BuyableTrait;

class Product extends Model implements Buyable
{
    use BuyableTrait;
}
Make sure you implement the getBuyableDescription and getBuyablePrice methods with the respective product description and product price.

Now you can add products to the cart.

use Treestoneit\ShoppingCart\Facades\Cart;

$product = Product::create(['name' => 'Pizza Slice', 'price' => 1.99]);
$quantity = 2;

Cart::add($product, $quantity);
To retrieve the cart contents:

Cart::content();
// or
Cart::items();
To retrieve the total:

Cart::subtotal();
You can update the quantity of an item in the cart. The first argument is the primary id of the related CartItem.

$item = Cart:content()->first();

Cart::update($item->id, $item->quantity + 5);
Or remove the item completely.

Cart::remove($item->id);
Options
To add item-specific options (such as size or color) to an item in the cart, first register available options in your Buyable instance.

class Product extends Model implements Buyable
{
    // ...
    
    public function getOptions(): array
    {
        return [
            'size' => ['18 inch', '36 inch'],
            'color' => ['white', 'blue', 'black'],
        ];
    }
}
Then you just pass an associative array as the third parameter of Cart::add.

Cart::add($product, 3, ['color' => 'white']);
Any invalid options will be silently removed from the array.

You can also add or change options of an item currently in the cart by calling Cart::updateOption.

$item = Cart:content()->first();

// Update a single option
Cart::updateOption($item->id, 'color', 'black');

// Update multiple options at once
Cart::updateOptions($item->id, [
    'color' => 'black',
    'size' => '36 inch',
]);
The options array will be available on the CartItem instance as $item->options.

Attaching to Users
You can attach a cart instance to a user, so that their cart from a previous session can be retrieved. Attaching a cart to a user is acheived by calling the attachTo method, passing in an instance of Illuminate\Contracts\Auth\Authenticatable.

class RegisterController
{
    /**
     * The user has been registered.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  mixed  $user
     * @return mixed
     */
    protected function registered(Request $request, $user)
    {
        Cart::attachTo($user);
    }
}
Then when the user logs in, you can call the loadUserCart method, again passing the user instance.

class LoginController
{
    /**
     * The user has been authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  mixed  $user
     * @return mixed
     */
    protected function authenticated(Request $request, $user)
    {
        Cart::loadUserCart($user);
    }
}
Dependency Injection
If you're not a facade person, you can use the container to inject the shopping cart instance by type-hinting the Treestoneit\ShoppingCart\CartManager class, or the Treestoneit\ShoppingCart\CartContract interface.

Tax
The shopping cart can calculate the total tax of the items in the cart. Just call

$rate = 13; // The tax rate as a percentage

Cart::tax($rate);
You can also set a default tax rate in the included config file.

// config/shopping-cart.php

    'tax' => [
        'rate' => 6,
    ],
Then just call Cart::tax without a parameter.

Cart::tax();
If some of your items have different tax rates applicable to them, or are tax-free, no problem. First modify the config file:

// config/shopping-cart.php

    'tax' => [
        'mode' => 'per-item',
    ],
Then, set the tax rate per item by implementing the Taxable interface and defining a getTaxRate method.

use Treestoneit\ShoppingCart\Taxable;

class Product extends Model implements Buyable, Taxable
{
    /**
     * Calculate the tax here based on a database column, or whatever you will.
     *
     * @return int|float
     */
    public function getTaxRate()
    {
        if ($this->tax_rate) {
            return $this->tax_rate;
        }

        if (! $this->taxable) {
            return 0;
        }

        return 8;
    }
Now your items will have their custom tax rate applied to them when calling Cart::tax.
