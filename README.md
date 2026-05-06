# ATINA JDE Connector Demo

This demo project showcases how to use the **ATINA JDE Mule Connector** to interact with JD Edwards EnterpriseOne through both Web Services (BSSV) and Business Functions (BSFN). It also includes a synchronization flow with Salesforce.

---

## Project Structure

- **demo-jde-config.xml**  
  Defines global connector configurations (with and without Web Service).
- **demo-jde-atina-connector.xml**  
  Exposes REST endpoints defined in the RAML specification and maps them to JDE operations.
- **demo-sync-customers.xml**  
  Provides a scheduler and synchronization mechanism between JDE and Salesforce.
- **demo-jde-atina-connector.raml**  
  RAML definition exposing the API endpoints.

---

## Configuration

In `demo-jde-config.xml` you will find two connector configurations:

- `JDE_Atina_Config_for_Web_Services` → Used when calling JDE Web Services (BSSV).  
- `JDE_Atina_Config_Non_WS_Operation` → Used when calling JDE BSFNs directly or batch processes.  

Credentials are externalized in `jde-credentials-{env}.properties`.

---

## API Endpoints (RAML)

### 1. Get Address Book (via WS)
**GET** `/v1/getAdressBookUsingWS?entityID={number}`  
Invokes the JDE Web Service `JP010000.AddressBookManager.getAddressBook`.

### 2. Get Address Book (via BSFN)
**GET** `/v1/getAdressBookUsingBSFN?entityID={number}`  
Invokes the BSFN `AddressBookMasterMBF`.

### 3. Create Customer (via BSFN)
**POST** `/v1/createCustomerUsingBSFNs`  
Creates a customer by combining BSFN `AddressBookMasterMBF` and `MBFCustomerMaster`.  
Example body:
```json
{
  "customerName": "John Doe",
  "customerAddress": "123 Main St",
  "customerCity": "Denver",
  "customerState": "CO",
  "paymentCurrency": "USD"
}
```

### 4. Submit UBE Process
**POST** `/v1/submitUBER01010Z`  
Submits a batch process (`R01010Z-ZJDE0001`) and retrieves job status.  
Example body:
```json
{
  "PO_ProcessMode": "0",
  "DataSelection_EDUS": "JDE"
}
```

### 5. Get Customers Created (via Events)
**GET** `/v1/getCustomerCreated?qtyRecords={number}`  
Captures customer creation events from JDE outbound interoperability tables.

---

## Error Handling

Each flow includes error handling for:
- Bad requests (400)  
- Resource not found (404)  
- Unsupported methods or media types (405, 415)  
- Not implemented operations (501)  

---

## Salesforce Synchronization (demo-sync-customers.xml)

This demo also shows how to synchronize JDE customer events with Salesforce Accounts.

1. **Scheduler** runs every 5 minutes.  
2. **Poll Events** captures new JDE customers.  
3. **VM Queue** ensures asynchronous processing.  
4. **Invoke WS** retrieves JDE Address Book details.  
5. **Salesforce Upsert** creates or updates the Account record in Salesforce.  
6. Deletes are handled by deactivating the Salesforce Account.


## Importing the Project into Anypoint Studio

To run this demo in **Anypoint Studio**, you need to import the project into your workspace:

1. Open **Anypoint Studio**.
2. Go to **File → Import → Anypoint Studio Project from File System**.
3. Browse to the project folder (for example: `demo-jde-connector`).
4. Click **Finish** to complete the import.

For more details, please refer to the official MuleSoft documentation:  
👉 [Import an existing Mule project into Anypoint Studio](https://docs.mulesoft.com/studio/import-export-packages)

### Updating Credentials

The demo requires valid **JDE** and **Salesforce** credentials.  
These values are stored in the properties file

---

## Running the Demo

1. Deploy the project to **Anypoint Studio** or **CloudHub**.  
2. Configure `jde-credentials-local.properties` with your JDE credentials.  
3. Call the API endpoints using Postman or your browser:
   - `http://localhost:8081/api/v1/getAdressBookUsingWS?entityID=123`
   - `http://localhost:8081/api/v1/getAdressBookUsingBSFN?entityID=123`
   - `http://localhost:8081/api/v1/createCustomerUsingBSFNs`
   - `http://localhost:8081/api/v1/submitUBER01010Z`
   - `http://localhost:8081/api/v1/getCustomerCreated?qtyRecords=10`

---

## Conclusion

This demo illustrates how the **ATINA JDE Connector** can:
- Invoke both JDE **Web Services** and **BSFNs**.  
- Handle **transactional operations** (commit/rollback).  
- Submit **UBE batch jobs**.  
- Capture **outbound events**.  
- Synchronize customers with **Salesforce**.  
