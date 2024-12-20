---
description: >-
  Explore New Era Caps's intricate inventory management strategies, from
  real-time POS sales updates to seamless integration with HotWax Commerce.
---

# Inventory

This section establishes the systems and their responsibilities in relation to the inventory life cycle.

## SAP
SAP is the true source of inventory for the company. The main bifurcations of inventory in SAP are to help separate inventory between the wholesale, physical retail, and eCommerce business.

As new inventory arrives at their physical warehouse, it is first incremented/received at the wholesale inventory location in SAP.

When the inventory is decided to be sold at either a retail location or eCommerce website, it is moved in SAP as an inventory transfer between the logical buckets.

If inventory is being sent to a retail location in Tokyo, then an inventory transfer will be raised from the wholesale warehouse location in SAP with a destination to the Tokyo store.

When the warehouse team fulfills the transfer order to the physical retail location in Tokyo, the transfer is moved from a “pending fulfillment” status to a “fulfilled/pending receipt” status in SAP. Once the store confirms receipt of all inventory, the transfer is marked as received at the store location. The inventory is, at this point, increased at the physical retail location in SAP.

The inventory level is incremented in Smaregi manually when inventory is received from the warehouse. There is no automatic communication from SAP about increased inventory.

## Smaregi
Smaregi is the POS system in New Era Caps retail locations. For the HotWax OMS, this system is the true record of accurate store inventory. Smaregi does not track available and on hand inventory levels for a product separately, so all inventory levels received from Smaregi should be accepted as on hand inventory.

If a product is not stocked at a store because it is sold out, its inventory level will be 0 or a negative number close to 0 to accurately represent the inventory position of that product in Smaregi.

### In Store Inventory
Every morning Smaregi imports a CSV into HotWax with the current inventory position at all retail locations for all products.

The Flagship middleware sends store inventory counts for products at retail locations to HotWax Commerce in a file based integration once a day. The Flagship middleware is responsible for converting the inventory file produced by Smaregi into a CSV format compatible with the native OMS Reset Inventory format.

Reset Inventory SFTP location:

```
/home/newera-uat-sftp/pos/morning_inventory_sync/staging/incoming
```
Hotwax OMS process the reset inventory file everyday at 5:30AM morning
```
Job Name: Import inventory reset
```

### POS Sales

Throughout the day, POS sale inventory reductions are sent to HotWax to keep inventory accurate in HotWax.

Flagship middleware is also responsible for posting inventory adjustments into the OMS to account for POS sales orders recorded in Smaregi. Flagship will use the OMS’s REST API endpoint for updating inventory sold by POS sales because it allows them to post updates in near real time helping maintain store inventory accuracy in the OMS, and by extension Shopify, to avoid over selling online.

Update Inventory API

```
https://<host>/api/service/updateInventoryByIdentification
```

{% hint style="info" %}
**API Docs** for the [Update Invetory API](\[udpateInvAPI]\(https://docs.hotwax.co/documents/integrate-with-hotwax/hotwax-commerce-api-and-data-feeds/inventory/update-inventory)) are a great place to learn more about how to use this API.
{% endhint %}

Other inventory deductions that happen in Smaregi that need to be posted to HotWax to ensure accurate store inventory:
- Customer calls in to reserve inventory which they’ll pickup later
- Inventory is shipped back to warehouse or another store
- Inventory is damaged or otherwise reduced manually

Each of these events should be sent to HotWax with a unique variance reason to help retain inventory traceability.


### Store fulfilled inventory

As order items are fulfilled from stores, the OMS reduces inventory in Smaregi that was used for store fulfillment. Traditionally the OMS would feed a list of shipped orders to the POS to ensure that it can maintain an accurate count of inventory in its system. New Era Caps found that this duration could actually be too long and cause retail operational issues and instead will depleted inventory in Smaregi POS as soon as orders are packed.

Store fulfilled orders SFTP

```
/home/newera-uat-sftp/pos/store_fulfilled_orders/outgoing
```

**Packed order rejection**

Once an order is packed by a store representative, New Era Caps will not allow the order to be unlocked, hence preventing any rejection and enabling them to accurately deplete store inventory at that time.

## Warehouse Inventory

New Era Caps uses a manual inventory upload procedure because their warehouse services also services their B2B business which does not flow through the OMS. The merchandising team manually splits inventory between the two channels and then uploads the inventory allotted to B2C sales manually into the OMS.

{% hint style="info" %}
**EXIM Link** This MDM is used to the update inventory in Hotwax [MDM Link](\[MDM]\(https://newera-uat.hotwax.io/commerce/control/ShopifyImportData?configId=IMP_INV_VAR_SHOPIFY)) .
{% endhint %}

## Shopify
Shopify’s sole responsibility is to capture orders based on the inventory available to it. As orders are captured it commits inventory to those orders to keep inventory levels accurate. All other inventory events happening at stores and warehouse are posted to Shopify from HotWax in frequent intervals of 5 minutes.

## HotWax
HotWax is the eCommerce inventory management system where the operations team will be able to get a unified view of their inventory position across retail locations and warehouse. All inventory movements made to warehouse inventory are made directly in HotWax, which syncs these changes to Shopify in as close to real time as possible.

### Posting inventory to Shopify

HotWax pushes available to sell inventory levels to Shopify. Because New Era Caps does not use Shopify POS in Japan, HotWax will map store facility to store location in Shopify.

All stores that participate in online sales will post their inventory to their respective location on Shopify. 

1. Delta sync for Stores and Warehouse seperately
2. Scheduled Restock
3. Store Inventory Reset

{% content-ref url="../salesOrder/omnichannelOrders.md" %}
[omnichannelOrders.md](../salesOrder/omnichannelOrders.md)
{% endcontent-ref %}
