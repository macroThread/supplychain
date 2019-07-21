**Note Delete all the assets and participarts before starting the testing, also delete all the existing code in script.js, queries.qry files and use the latest code**
queries.qry -> https://github.com/macroThread/supplychain/blob/master/queries.qry
<br />
script.js ->  https://github.com/macroThread/supplychain/blob/master/script.js
<br />
mode.cto -> https://github.com/macroThread/supplychain/blob/master/model.cto
<br />

Note: Most of the inputs are actually filled in by default by the playground UI. <br /> <br /> 
Steps, imagine we are procuring wheat, manufacturing maggi, sending it to distributor: <br /><br />
1) First  in the test tab, **run the setUpDemo transaction**, it will create all the actors(**Distributor, Manufacturer, Retailer, Supplier**). check in the left side for various actors in system, check the data and the identifiers, i.e  **gs1CompanyPrefix**.
<br /><br />

2) **Now create a new commodity in the assets sub section in the left side of the page**, notice the owner field, i had set the value to the supplier@email.com, who is created during step 1, while we ran the **setUpDemo transaction**. Other fields I chose random values or I just let the values suggested by the playground.
<br />

```

{
  "$class": "org.logistics.testnet.Commodity",
  "type": "FOOD",
  "GTIN": "5477",
  "name": "Wheat",
  "description": "wheat flour",
  "itemCondition": {
    "$class": "org.logistics.testnet.ItemCondition",
    "conditionDescription": "very good",
    "status": "GOOD"
  },
  "owner": "resource:org.logistics.testnet.Supplier#supplier@email.com"
}

```

<br />
3) **Now create the contract and shipment using CreateShipmentAndContract** after this step check the OrderContract and ShipmentBacth assets in the left side of the page, new ones will be created <br /><br />

**Notice the owner and buyer field**, i had plugged in values from existing supplier and manufacturer records, created by **setupDemo** txn above in **step 1** <br /><br />
**Notice the expectedArrivalLocation** (manufacturer’s address UK )and **location**(suppliers address -  USA). <br /><br />
**Notice assetExchanged** - filled  in with the commodity id created earlier in **step 2**<br /><br />

```
{
    "$class": "org.logistics.testnet.CreateShipmentAndContract",
    "shipmentId": "1",
    "trackingNumber": "1",
    "message": "supplying wheat",
    "status": "WAITING",
    "location": {
        "$class": "org.logistics.testnet.Location",
        "globalLN": "1",
        "address": {
            "$class": "org.logistics.testnet.Address",
            "country": "USA"
        }
    },
    "owner": "resource:org.logistics.testnet.Supplier#supplier@email.com",
    "assetExchanged": [
        "resource:org.logistics.testnet.Commodity#5477"
    ],
    "orderId": "1",
    "buyer": "resource:org.logistics.testnet.Manufacturer#manufacturer@email.com",
    "expectedArrivalLocation": {
        "$class": "org.logistics.testnet.Location",
        "globalLN": "2",
        "address": {
            "$class": "org.logistics.testnet.Address",
            "country": "UK"
        }
    },
    "payOnArrival": false,
    "arrivalDateTime": "2019-07-30T04:43:05.693Z",
    "paymentPrice": 0
}
```
<br /><br />

Note: I deleted all the stuff and then ran till step 3, before running step 4

4) **updateShipment tranasaction** after creating the contract and shipment in previous step, we will now update the shipment 

```
{
  "$class": "org.logistics.testnet.UpdateShipment",
  "status": "SHIPPED_IN_TRANSIT",
  "newLocation": {
    "$class": "org.logistics.testnet.Location",
    "globalLN": "3",
    "address": {
      "$class": "org.logistics.testnet.Address",
      "country": "Ireland"
    }
  },
  "message": "reached ireland",
  "shipment": "resource:org.logistics.testnet.ShipmentBatch#1"
}
```
<br /><br />
Changing the location to a country between USA and UK, let's say Ireland, with **globalLN** to be 3, and status **SHIPPED_IN_TRANSIT**
Notice that the **shipment** field is filled with the shipmentId we supplied in previous transaction. 
Also notice that the value supplied in the **status** field is from  **ShipmentStatus enum** in model.cto file.

After running the transaction, notice that the location inside the **ShipmentBatch** asset in the left side of the page, it will be updated to Ireland.

<br /><br />


5) Now to simulate that the shipment reached the final location UK, as we have have specified in the expectedArrivalLocation, run the step 4 again but
this time using the **globalLN** to 2 and **country** to 2, **status** to DELIVERED

```
{
  "$class": "org.logistics.testnet.UpdateShipment",
  "status": "DELIVERED",
  "newLocation": {
    "$class": "org.logistics.testnet.Location",
    "globalLN": "2",
    "address": {
      "$class": "org.logistics.testnet.Address",
      "country": "UK"
    }
  },
  "message": "reached UK",
  "shipment": "resource:org.logistics.testnet.ShipmentBatch#1"
}
```

After running the transaction, notice that the location inside the **ShipmentBatch** asset in the left side of the page, it will be updated to UK.
The owner of the commodity is still the supplier, even after product is delivered. So we need to run **TransferCommodityPossession** to change thwe ownership.

If we also fill the **newHolder** property(it is optional, so i did not fill) in  above UpdateShipment txn in step 5, it will change the owner of the commodity.

<br /><br />
6) Now we run TransferCommodityPossession to change the owner of the commodity from **supplier@email.com** to **manufacturer@email.com**, we have used these values in **CreateShipmentAndContract** in step 3
For this transaction to run I add a new query in **queries.qry** file and made a small change in **script.js** file inside **isAnyAssetBeingShipped** function

```
{
  "$class": "org.logistics.testnet.TransferCommodityPossession",
  "commodity": "resource:org.logistics.testnet.Commodity#5477",
  "newOwner": "resource:org.logistics.testnet.Manufacturer#manufacturer@email.com"
}
```

After this step notice the owner inside the commodity asset in the left side of the page. it will change to **manufacturer@email.com**
<br /><br />

7) Need to test **TransformCommodities**, which can consume some raw materials(commodities) and create new products(can be called commodities again). This step is a simulation of manufaturing of products from raw materials in supply chain.


```
{
  "$class": "org.logistics.testnet.TransformCommodities",
  "commoditiesToBeConsumed": ["resource:org.logistics.testnet.Commodity#5477"],
  "commoditiesToBeCreated": [{
  "$class": "org.logistics.testnet.Commodity",
  "type": "FOOD",
  "GTIN": "5478",
  "name": "maggi",
  "description": "maggi noodles",
  "itemCondition": {
    "$class": "org.logistics.testnet.ItemCondition",
    "conditionDescription": "very good",
    "status": "GOOD"
  },
  "owner": "resource:org.logistics.testnet.Manufacturer#manufacturer@email.com"
}]
}
```

After this step the old commodity gets deleted and new commodity is created in the assets section in the left side of the page
