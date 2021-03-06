# Upgrading to Commerce 3

::: warning
If you’re upgrading from Commerce 1, see the [Changes in Commerce 2](https://docs.craftcms.com/commerce/v2/changes-in-commerce-2.html) and upgrade to the latest Commerce 2 version before upgrading to Commerce 3.
:::

## Preparing for the Upgrade

Before you begin, make sure that:

1. You’ve reviewed the [changes in Commerce 3](https://github.com/craftcms/commerce/blob/master/CHANGELOG.md#300---2020-01-28)
2. Your site’s running at least **Craft 3.4** and **the latest version of Commerce 2** (2.2.17)
3. Your **database is backed up** in case everything goes horribly wrong

Once you’ve completed these steps, you’re ready continue.

When upgrading from Commerce 2 to Commerce 3, the following changes may be important depending on how you’ve set up your project.

## Order Emails

Order notification emails are now sent via a queue job, so running a [queue worker as a daemon](https://nystudio107.com/blog/robust-queue-job-handling-in-craft-cms) is highly recommended to avoid customer email notification delays.

Previously emails would be generated during customer checkout, which could cause the order completion page to take a prolonged time to display (especially with PDF generation involved). This change gives your customers a better checkout experience.

No changes are needed for emails to continue to work, but ensuring your queue is working correctly will ensure everything goes smoothly.


## Edit Order page

Plugins and modules that modify the Edit Order page are likely to break with this update as the page is now a [Vue.js](https://vuejs.org/) application. 
The same Twig template hooks are still available, but inserting into the part of the DOM controlled by Vue.js will not work.


## Data tables

All data tables throughout the control panel use the new Craft 3.4 Vue.js-based data table, so any extensions of those old HTML tables are likely to break.


## Permissions

We have added the “Edit orders” and “Delete orders” user permissions, but users with the existing “Manage orders” permission will not automatically get these new permissions, so updating those users and user groups would be required.


## Cart merging

Automatic cart merging has been removed.

The cart is still retrieved from the front end templates using `craft.commerce.carts.cart` in your templates, but the optional `mergeCarts` param has been removed, and it is no longer possible to automicatically merge previous carts of the current user. 

We now recommend the customer manually adds items from the [previous carts to their current cart](adding-to-and-updating-the-cart.md#restoring-previous-cart-contents). An example of this is in the example templates.

Merging carts as manual process is better since the customer can decide what to do with any validation issues like maximum quanity rules. The previous merge feature would just fail to merge correctly with no error messages. 

This change is also mitigated by the fact that the previous cart of the current user is now loaded as the current cart when calling `craft.commerce.carts.cart` automatically.

## Twig template breaking changes

Use the table below to update each breaking change in your Twig templates.

| Old                                       | New                                                                 |
| ----------------------------------------- | ------------------------------------------------------------------- |
| `craft.commerce.carts.cart(true, true)`   | `craft.commerce.carts.cart(true)`                                   |
| `craft.commerce.carts.cart(false, true)`  | `craft.commerce.carts.cart(false)`                                  |
| `craft.commerce.availableShippingMethods` | `cart.availableShippingMethod`                                      |
| `craft.commerce.cart`                     | `craft.commerce.carts.cart`                                         |
| `craft.commerce.countriesList`            | `craft.commerce.countries.allCountriesAsList`                       |
| `craft.commerce.customer`                 | `craft.commerce.customers.customer`                                 |
| `craft.commerce.discountByCode`           | `craft.commerce.discounts.discountByCode`                           |
| `craft.commerce.primaryPaymentCurrency`   | `craft.commerce.paymentCurrencies.primaryPaymentCurrency`           |
| `craft.commerce.statesArray`              | `craft.commerce.states.allStatesAsList`                             |
| `craft.commerce.states.allStatesAsList`          | `craft.commerce.states.getAllEnabledStatesAsListGroupedByCountryId` |

## Form Action Changes

| Old                                    | New                         | Docs                                                                        |
| -------------------------------------- | --------------------------- | --------------------------------------------------------------------------- |
| `commerce/cart/remove-line-item`       | `commerce/cart/update-cart` | [Updating the Cart](adding-to-and-updating-the-cart.md#updating-line-items) |
| `commerce/cart/update-line-item`       | `commerce/cart/update-cart` | [Updating the Cart](adding-to-and-updating-the-cart.md#updating-line-items) |
| `commerce/cart/remove-all-line-items`  | `commerce/cart/update-cart` | [Updating the Cart](adding-to-and-updating-the-cart.md#updating-line-items) |

## Event Changes

| Old                                                          | New                                                        |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| `craft\commerce\models\Address::EVENT_REGISTER_ADDRESS_VALIDATION_RULES` | `craft\base\Model::EVENT_DEFINE_RULES`         |
| `craft\commerce\services\Reports::EVENT_BEFORE_GENERATE_EXPORT` | `craft\base\Element::EVENT_REGISTER_EXPORTERS`          |

