# Scan&Shop in Tungsten Checkouts
Since version 1.03, Tungsten natively supports Scan&Shop, which means it can process transaction data sent via BindableFunction, handle errors, and send a response.

## Sending transaction data to the checkout
In this section, we will explain how to send transaction data to the Tungsten Self Checkout system using the Scan&Shop feature. The example provided demonstrates how to structure the data for a typical transaction, including the items being purchased and related information.

Example code:
```
workspace["SwiftRetail | Tungsten Self Checkout System"].Event.Communication:Invoke("Scan&Shop", {
    InterventionTitle = "My Scan&Shop",
	Manufacturer = "Constellation",
	DesiredLane = 1,
	Player = Player,
	Items = {
        [1] = {
			ProductName = "Beans",
			ProductPrice = 1.29,
			ClubcardPrice = 1.19,
			Units = 1,
			AgeRestricted = false,
			ImageID = nil,
			ItemInstances = {BeansTool},
		}
    }
})
```
### Explanation
1. **InterventionTitle**: This field specifies the header of the intervention displayed on the screen when the Scan&Shop transaction is being processed. ```string```

2. **Manufacturer**: This field is used to define the manufacturer of the checkout system. Here, it is set to "Constellation", indicating that the system is developed by Constellation. ```string```

3. **DesiredLane**: This represents the specific checkout number where the transaction is going to be processed. For this example, the transaction will be processed in lane 1. ```number```

4. **Player**: This refers to the player who is handling the transaction. ```player```

5. **Items**: This field contains all the items being purchased in the transaction. Each item is represented by a table, and the fields inside the item table describe various properties of the product: ```table```

    - *ProductName*: Required ```string```
    - *ProductPrice*: Required ```number```
    - *ClubcardPrice*: Optional ```number```
    - *Units*: Required ```number``` 
    - *AgeRestricted*: Required ```boolean``` 
    - *ImageID*: Optional```string``` 
    - *ItemInstances*: Required (Can contain multiple instances, may be empty) ```table``` 

## Handling responses
In this section we will describe how Tungsten checkout may respond to a Scan&Shop request.

### Possible responses
1. ```"Error (Checkout not found)"```

    This means that the checkout with provided ```DesiredLane``` number cannot be found.

2. ```"Error (Checkout is busy)```

    This response occurs when the system is currently busy and unable to process the transaction. 

3. ```"Error (Checkout offline)"```

    This indicates that the checkout has been found but the system is offline and cannot process transactions.

4. ```"Error (Cannot start transaction)"```

    This occurs when the system cannot start the transaction.

5. ```"Error (Basket is empty)"```

    This response occurs when the system receives an empty item list (data.Items does not exist or is empty).

6. ```"Error (Error while transferring data to checkout)"```

    The system encountered an error while processing the items or sending data from Scan&Shop to the checkout terminal. An error during data processing may result in intervention and cancellation of the entire transaction.

7. ```"Error (Timeout while communicating with checkout)"```

    The checkout did not respond to the sent request.
    
8. ```true (boolean)```

    Successfully transferred all transaction data to the checkout.
