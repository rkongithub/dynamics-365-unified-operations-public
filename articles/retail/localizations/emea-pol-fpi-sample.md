---
# required metadata

title: Fiscal printer integration sample for Poland
description: This topic provides an overview of the fiscal integration sample for Poland.
author: josaw
manager: annbe
ms.date: 02/01/2019
ms.topic: article
ms.prod: 
ms.service: dynamics-365-retail
ms.technology: 

# optional metadata

ms.search.form: RetailFunctionalityProfile, RetailFormLayout, RetailParameters
audience: Application User
# ms.devlang: 
ms.reviewer: josaw
ms.search.scope: Core, Operations, Retail
# ms.tgt_pltfrm: 
# ms.custom: 
ms.search.region: Poland
ms.search.industry: Retail
ms.author: v-dmpere
ms.search.validFrom: 2019-2-1
ms.dyn365.ops.version: 10.0.1

---

# Fiscal printer integration sample for Poland

[!include[banner](../includes/banner.md)]

## Introduction

The Microsoft Dynamics 365 for Retail functionality for Poland includes a sample integration of the point of sale (POS) with a fiscal printer. The sample extends the [fiscal integration functionality](fiscal-integration-for-retail-channel.md) and supports the POSNET THERMAL HD 2.02 protocol for fiscal printers from [Posnet Polska S.A.](http://www.posnet.com.pl) The sample enables communication with a fiscal printer that is connected via a COM port by using a native software driver. It was implemented and tested by using a software emulator that Posnet provided for the Posnet Thermal HD FV EJ fiscal printer. The sample is provided in the form of source code and is part of the Retail software development kit (SDK).

Microsoft doesn't release any hardware, software, or documentation from Posnet. For information about how to get the fiscal printer and operate it, contact [Posnet Polska S.A.](http://www.posnet.com.pl)

## Scenarios

The following scenarios are covered by the fiscal printer integration sample for Poland:

- Sales scenarios:

    - Print a fiscal receipt for cash-and-carry sales and returns.
    - Capture a response from the fiscal printer, and store it in the channel database.
    - Taxes:

        - Map to the fiscal printer's tax codes (departments).
        - Transfer mapped tax data to the fiscal printer.

    - Payments:

        - Map to the fiscal printer's methods of payment.
        - Print payments on a fiscal receipt.
        - Print change information.

    - Print line discounts.
    - Gift cards:

        - Exclude an issued/re-charged gift card line from a fiscal receipt for a sale.
        - Print a payment that uses a gift card as a regular method of payment.

    - Print fiscal receipts for customer order operations:

        - A fiscal receipt isn't printed for a customer order deposit.
        - Print a fiscal receipt for carry-out lines of a hybrid customer order.
        - Print a fiscal receipt for the pickup operation for a customer order.
        - Print a fiscal receipt for a return order.

- End of day statements (fiscal X and fiscal Z reports).
- Error handling, such as the following options:

    - Retry fiscal registration if a retry is possible, such as if the fiscal printer isn't connected, isn't ready, or isn't responding, the printer is out of paper, or there is a paper jam.
    - Postpone fiscal registration.
    - Skip fiscal registration, or mark the transaction as registered, and include info codes to capture the reason for the failure and additional information.

### Default data mapping

The following default data mapping is included in the fiscal document provider configuration that is provided as part of the fiscal integration sample:

- Value-added tax (VAT) rates mapping:

    *0 : 23.00 ; 1 : 8.00 ; 2 : 5.00 ; 3 : 0.00*

- Tender type mapping:

    *0 : 0 ; 1 : 0 ; 2 : 2 ; 3 : 2 ; 4 : 0 ; 5 : 0 ; 6 : 0 ; 7 : 2 ; 8 : 0*

### Handling gift cards

The fiscal printer integration sample implements the following rules that are related to gift cards:

- Exclude sales lines that are related to the *Issue gift card* and *Add to gift card* operations from the fiscal receipt.
- Don't print a fiscal receipt if it consists only of gift card lines.
- Deduct the total amount of gift cards that are issued or re-charged in a transaction from payment lines of the fiscal receipt.
- Save calculated adjustments of payment lines in the channel database with a reference to a corresponding fiscal transaction.
- Payment by gift card is considered a regular payment.

### Handling customer deposits and customer order deposits

The fiscal printer integration sample implements the following rules that are related to customer deposits and customer order deposits:

- Don't print a fiscal receipt if a transaction is a customer deposit.
- Don't print a fiscal receipt if a transaction contains only a customer order deposit or a customer order deposit refund.
- Print the amount of the previously paid deposit on a fiscal receipt for a customer order pickup operation.
- Deduct the customer order deposit amount from payment lines when a hybrid customer order is created.
- Save calculated adjustments of payment lines in the channel database with a reference to a fiscal transaction for a hybrid customer order.

## Set up Retail for Poland

### Enable extensions

##### Commerce runtime extension components

The Commerce runtime (CRT) extension components are included in the Retail SDK. To complete the following procedures, open the CRT solution, **CommerceRuntimeSamples.sln**, under **RetailSdk\\SampleExtensions\\CommerceRuntime**.

1. Find the **Runtime.Extensions.DocumentProvider.PosnetSample** project, and build it.
2. In the **Extensions.DocumentProvider.PosnetSample\\bin\\Debug** folder, find the **Contoso.Commerce.Runtime.Extensions.DocumentProvider.PosnetSample.dll** assembly file.
3. Copy the assembly file to the CRT extension folder:

    - **Retail Server:** Copy the assembly to the **\\bin\\ext** folder under the Microsoft Internet Information Services (IIS) Retail Server site location.
    - **Local CRT on Modern POS:** Copy the assembly to the **\\ext** folder under the local CRT client broker location.

4. Find the extensions configuration file for CRT:

    - **Retail Server:** The file is named **commerceruntime.ext.config**, and it's in the bin\\ext folder under the IIS Retail Server site location.
    - **Local CRT on Modern POS:** The file is named **CommerceRuntime.MPOSOffline.Ext.config**, and it's under the local CRT client broker location.

5. Register the CRT change in the extension's configuration file. Add **source="assembly" value="Contoso.Commerce.Runtime.Extensions.DocumentProvider.PosnetSample"**.
6. Restart the Retail service.

    - **Retail Server:** Restart the Retail service site from IIS Manager.
    - **Client broker:** End the **dllhost.exe** process in Task Manager, and then restart Modern POS.

##### Hardware station extension components

The Hardware station extension components are included in the Retail SDK. To complete the following procedures, open the Hardware Station solution, **HardwareStationSamples.sln**, under **RetailSdk\\SampleExtensions\\HardwareStation**.

1. Find the **Extension.PosnetThermalFVFiscalPrinterSample** project, and build it.
2. In the **Extension.PosnetThermalFVFiscalPrinterSample\\bin\\Debug** folder, find the **Contoso.Commerce.HardwareStation.PosnetThermalFVFiscalPrinterSample.dll** assembly file.
3. Copy the files to a deployed Hardware station machine:

    - **Remote Hardware station:** Copy the files to the **bin** folder under the IIS Hardware station site location. Copy the printer driver libraries (**libposcmbth.dll**, **libcmbth\_serial.dll**, and **cmbth\_pl.lng**).

4. Find the configuration file for the Hardware station's extensions. The file is named **HardwareStation.Extension.config**:

    - **Remote Hardware station:** The file is located under the IIS Hardware station site location.

5. Add the following section to the **composition** section of the config file.

    ```
    <add source="assembly" value="Contoso.Commerce.HardwareStation.PosnetThermalFVFiscalPrinterSample" />
    ```

6. Restart the Hardware station service:

	- **Remote Hardware station:** Restart the Hardware station site from IIS Manager.

### Set up the registration process

To enable the registration process, follow these steps to set up Retail Headquarters. For more details, see [Set up a fiscal registration process](setting-up-fiscal-integration-for-retail-channel.md#set-up-a-fiscal-registration-process).

1. Go to **Retail \> Channel Setup \> Fiscal Integration \> Fiscal Connectors**. Import the configuration from **RetailSdk\\SampleExtensions\\HardwareStation\\Extension.Posnet.ThermalDeviceSample\\Configuration\\ConnectorConnectorPosnetThermalFVEJ.xml**.
2. Go to **Retail \> Channel Setup \> Fiscal Integration \> Fiscal Document providers**. Import the configuration from **RetailSdk\\SampleExtensions\\CommerceRuntime\\Extension.DocumentProvider.PosnetSample\\Configuration\\DocumentProviderPosnetSample.xml**.
3. Go to **Retail \> Channel Setup \> Fiscal Integration \> Connector Technical profiles**. Create a new profile, and select the loaded connector from the earlier step. Update connection settings if an update is required.
4. Go to **Retail \> Channel Setup \> Fiscal Integration \> Connector Functional profiles**. Create a new profile, and select the loaded connector and document provider from the earlier steps. Update data mapping settings, if an update is required.
5. Go to **Retail \> Channel Setup \> Fiscal Integration \> Connector Functional group**. Create a new group, and select the connector functional profile from the earlier step.
6. Go to **Retail \> Channel Setup \> Fiscal Integration \> Registration process**. Create a new process, and select the connector functional group from the earlier step.
7. Go to **Retail \> Channel setup \> POS setup \> POS profiles \> Functionality profiles**. Open the functionality profile that is linked to the store where the registration process should be activated. On the **Fiscal registration process** FastTab, select the registration process that was created earlier.
8. Go to **Retail \> Channel setup \> POS setup \> POS profiles \> Hardware profiles**. Open the hardware profile that is linked to the Hardware station that the fiscal printer will be connected to. On the **Fiscal peripherals** FastTab, select the connector technical profile.
9. Open the distribution schedule (**Retail \> Retail IT \> Distribution schedule**), and select job **1070** to transfer data to the channel database.

## Commerce runtime extension design

The purpose of the extension (document provider) is to generate printer-specific documents and handle responses from the fiscal printer.

The Commerce runtime extension is **Commerce.Runtime.DocumentProvider.PosnetSample.DocumentProviderPosnetProtocol**. This extension generates the set of printer-specific commands that are defined by POSNET specification 19-3678, in JavaScript Object Notation (JSON) format.

For more details about the design of the fiscal integration solution, see [Fiscal registration process and fiscal integration samples for fiscal devices](fiscal-integration-for-retail-channel.md#fiscal-registration-process-and-fiscal-integration-samples-for-fiscal-devices).

### Request handler
	
The **DocumentProviderPosnetProtocol** request handler is the entry point for the request to generate documents from the fiscal printer.

The handler is inherited from the **INamedRequestHandler** interface. The **HandlerName** method is responsible for returning the name of the handler. The handler name should match the connector document provider name that is specified in Retail Headquarters.

The connector supports the following requests:

- **GetFiscalDocumentDocumentProviderRequest** – This request contains information about what document should be generated. It returns a printer-specific document that should be registered in the fiscal printer.
- **GetSupportedRegistrableEventsDocumentProviderRequest** – This request returns the list of events to subscribe to. Currently, the following events are supported: sales, printing X report, and printing Z report.
- **SaveFiscalRegistrationResultDocumentProviderRequest** – This request saves the response from the printer.

### Configuration

The configuration file is found in the **Configuration** folder of the extension project. The purpose of the file is to enable configuration of settings for the document provider from Retail Headquarters. The file format aligns fiscal integration configuration requirements. The following settings have been added:

- VAT rates mapping
- Tender type mapping
- Deposit payment type

## Hardware station extension design

The purpose of the extension (connector) is to communicate with the fiscal printer.

The Hardware station extension is **Commerce.HardwareStation.PosnetThermalFVFiscalPrinterSample.FiscalPrinterHandler**. This extension submits commands that the Commerce runtime extension generates to the fiscal printer, by calling the functions of the POSNET driver that is provided by the manufacturer. It also handles device errors.

### Request handler

The **FiscalPrinterHandler** request handler is the entry point for handling the request to the fiscal peripheral device.

The handler is inherited from the **INamedRequestHandler** interface. The **HandlerName** method is responsible for returning the name of the handler. The handler name should match the fiscal connector name that is specified in Retail Headquarters.

The connector supports the following requests:

- **SubmitDocumentFiscalDeviceRequest** – This request sends documents to printers and returns the response from the fiscal printer.
- **IsReadyFiscalDeviceRequest** – This request is used for a health check of the device.
- **InitializeFiscalDeviceRequest** – This request is used for printer initialization.

### Configuration

The configuration file is found in the **Configuration** folder of the extension project. The purpose of the file is to enable configuration of settings for the connector provider from Retail Headquarters. The file format aligns fiscal integration configuration requirements. The following settings have been added:

- **Connection string** – This string describes the details of the connection to the device in a format that is supported by the driver. For details, see the POSNET driver documentation.
- **Date and time synchronization** – This setting indicates whether you must sync the date and time of the printer with the connected Hardware station. The date and time of the printer will be synced with the Hardware station time.
- **Device timeout** – The amount of time, in milliseconds, that the driver will wait for a response from the device. For details, see the POSNET driver documentation.

## Limitations of the sample

- The fiscal printer supports only scenarios where sales tax is included in the price. Therefore, the **Price include sales tax** option must be set to **Yes** for both retail stores and customers.
- Daily reports (fiscal X and fiscal Z) are printed by using the embedded *Shift report* format.
- Printing a barcode on fiscal receipts is considered a potential customization, because this feature isn't supported in the embedded formats and can be implemented only through using the customizable **Super-format** report.
- Mixed transactions aren't supported by the fiscal printer. The **Prohibit mixing sales and returns in one receipt** option should be set to **Yes** in POS functionality profiles.
