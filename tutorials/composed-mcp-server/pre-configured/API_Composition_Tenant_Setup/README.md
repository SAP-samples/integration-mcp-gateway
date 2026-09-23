# SAP BTP Destinations Setup

>[!NOTE]
>**This section has already been pre-configured for you, and the details provided are for your information and learning purpose only.**

## Prerequisites

- **SAP BTP subaccount** in the Cloud Foundry environment with enabled Cloud Foundry
- **SAP Integration Suite** tenant, and the `Integration_Provisioner` role collection assigned to the user
- **API Composition** capability activated on the SAP Integration Suite and `Graph.KeyUser` role collection assigned to the user

For reference on how you can set the SAP Integration Suite trial tenant and enable the API Composition capability, see:

- [Check BTP regions where the API Composition capability is available](https://me.sap.com/notes/3338820)
- [Create an SAP BTP Trial Account](https://developers.sap.com/tutorials/hcp-create-trial-account.html)
- [Set up SAP Integration Suite on Trial](https://developers.sap.com/tutorials/cp-starter-isuite-onboard-subscribe.html)
- [Activate API Composition on SAP Integration Suite](https://help.sap.com/docs/api-composition/isuite-api-composition/initial-setup?locale=en-US#2.-activate-api-composition-on-sap-integration-suite)

Refer to the [API Composition Initial Setup](https://help.sap.com/docs/api-composition/isuite-api-composition/initial-setup) guide for full details.

---

## SAP BTP Destinations

The Business Data Graph used in this workshop composes data from four different backend systems APIs — 
- location risk,
- material stock,
- plant details, and
- business partner (supplier) data.

To make these systems accessible to SAP Integration Suite, four SAP BTP Destinations (HTTP) are already configured in the SAP BTP subaccount. Each destination points to one of the publicly hosted mock APIs provided for this workshop. All four mock APIs are hosted at `https://api-composition-plant-material-risk.cfapps.eu20-002.hana.ondemand.com` and are publicly accessible — no authentication is required.

| Destination Name | API Path | Represents |
| :--- | :--- | :--- |
| `demo_LocationRisk` | `https://api-composition-plant-material-risk.cfapps.eu20-002.hana.ondemand.com/location-risk/` | Location risk data (Everstream mock API) |
| `demo_S4_API_MATERIAL_STOCK_SRV` | `https://api-composition-plant-material-risk.cfapps.eu20-002.hana.ondemand.com/material-stock/` | Material stock levels (S/4HANA mock API) |
| `demo_S4_sap-s4-ce-plant-0001-v1` | `https://api-composition-plant-material-risk.cfapps.eu20-002.hana.ondemand.com/plant/` | Plant details (S/4HANA mock API) |
| `s4hana_api_business-partner` | `https://api-composition-plant-material-risk.cfapps.eu20-002.hana.ondemand.com/business-partner/` | Supplier / Business Partner data (S/4HANA mock API) |

1. Destinations are managed in the SAP BTP Cockpit. To navigate there from SAP Integration Suite, we need to click on the **grid icon (⠿)** in the top-right navigation bar and select **SAP BTP Cockpit**.

   ![SAP Integration Suite — grid icon menu with SAP BTP Cockpit selected](./assets/ex1-prereq-navigate-btp-cockpit.png)

3. In the BTP Cockpit, we go to the subaccount and navigate to **Connectivity → Destinations** in the left sidebar. All four destinations are created and listed here and are ready for use.

   ![BTP Cockpit — Connectivity → Destinations list showing all four configured destinations](./assets/ex1-prereq-destinations-list.png)

3. Each destination is created with the following configurations. The `Type` is set to **HTTP** and `Proxy Type` to **Internet** since these are external mock APIs. Authentication is set to **NoAuthentication** as the mock APIs are publicly accessible.

   | Field | Value |
   | :--- | :--- |
   | Name | *(destination name from the table above)* |
   | Type | **`HTTP`** |
   | Description | This destination points to a publicly accessible *(corresponding mock API name)* mock API |
   | Proxy Type | **`Internet`** |
   | URL | *(corresponding mock API URL)* |
   | Authentication | **`NoAuthentication`** |

   An additional property is added to each destination to make it visible within the API Composition capability. Without this property, the destination will not appear as an available data source when configuring the Business Data Graph.

   | Key | Value |
   | :--- | :--- |
   | `IntegrationCell.Include` | **`true`** |

   The screenshot below shows the completed destination form for `demo_LocationRisk` as a reference example. The same structure is applied to all four destinations, with the respective name and URL substituted.

   ![Destination form — completed example for demo_LocationRisk](./assets/ex1-prereq-create-destination.png)
