# Myket In-App Billing (IAB) Unity Plugin Sample

[![Unity](https://img.shields.io/badge/Unity-6000+-black?logo=unity)](https://unity.com/)
[![Platform](https://img.shields.io/badge/Platform-Android-green?logo=android)](https://developer.android.com/)
[![SDK Version](https://img.shields.io/badge/Billing%20SDK-v2.2-blue)](https://myket.ir/)
[![Java Compatibility](https://img.shields.io/badge/Java-17-orange?logo=openjdk)](https://openjdk.org/)

An official sample project demonstrating how to integrate and use the **Myket In-App Billing (IAB) SDK** in Unity games and applications targeting Android.

This sample showcases a full implementation of in-app products, subscriptions, inventory queries, consumption workflows, and event handling using Unity's modern Custom Gradle templates.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Quick Start: Running the Sample](#quick-start-running-the-sample)
- [Integrating into Your Own Project](#integrating-into-your-own-project)
  - [1. Import Plugin Assets](#1-import-plugin-assets)
  - [2. Configure Custom Gradle Templates](#2-configure-custom-gradle-templates)
  - [3. Configure ProGuard / R8 Rules](#3-configure-proguard--r8-rules)
- [Code Examples & API Guide](#code-examples--api-guide)
  - [Initialization](#initialization)
  - [Event Handling](#event-handling)
  - [Querying SKUs and Purchases](#querying-skus-and-purchases)
  - [Purchasing Products](#purchasing-products)
  - [Consuming Purchases](#consuming-purchases)
  - [Subscriptions Support](#subscriptions-support)
- [Debugging & Troubleshooting](#debugging--troubleshooting)
- [Support & Resources](#support--resources)

---

## Features

- **Consumable & Non-Consumable In-App Purchases:** Complete flow for standard products.
- **Subscription Support:** Support for monthly, annual, and recurring billing.
- **Inventory & Purchase Recovery:** Query purchase history and product information.
- **Consumption Management:** API for consuming single or multiple items in batch.
- **Event-Driven Architecture:** Clean C# events via `IABEventManager` for asynchronous callbacks.
- **Modern Gradle Workflow:** Native dependency resolution through Myket Maven repository using Unity's `settingsTemplate.gradle` and `launcherTemplate.gradle`.
- **R8 / ProGuard Optimization:** Pre-configured keep rules for Android release builds.

---

## Prerequisites

- **Unity Editor:** Unity 6 (6000.x) with **Android Build Support** installed.
- **JDK:** Java 17 compatible JDK.
- **Test Device:** A physical Android device with the **[Myket Application](https://myket.ir/)** installed and signed in.
- **Myket Developer Account:** A registered developer account on the [Myket Developers Portal](https://developer.myket.ir/) with:
  - Your application registered with a matching Package Name.
  - In-app products / subscriptions created.
  - Your developer Public Key available.

> [!NOTE]
> In-app billing requires the **Myket** store app to be installed and active on the device during testing. The plugin communicates with Myket's billing service via Android IPC (AIDL).

---

## Project Structure

```text
myket-billing-unity-sample/
├── Assets/
│   ├── MyketIAB/                         # Core Myket IAB C# Plugin
│   │   ├── MyketIAB.cs                   # Main native bridge API
│   │   ├── MyketIABEventManager.cs       # Event dispatcher (C# Actions)
│   │   ├── MyketPurchase.cs              # Purchase data model
│   │   ├── MyketSkuInfo.cs               # SKU/Product metadata model
│   │   ├── AbstractManager.cs            # Unity-Android message bridge base
│   │   ├── ActionExtensions.cs           # Utility extensions for events
│   │   ├── SimpleJSON.cs                 # Lightweight JSON parser
│   │   └── Demo/                         # Sample Demo Scene & UI
│   │       ├── IABTestScene.unity        # Interactive demo scene
│   │       ├── IABTestUI.cs              # OnGUI test interface
│   │       └── MyketIABEventListener.cs  # Sample event listener implementation
│   └── Plugins/
│       └── Android/                      # Android-specific build configuration
│           ├── AndroidManifest.xml       # Custom Android manifest
│           ├── launcherTemplate.gradle   # Application-level Gradle config & dependencies
│           ├── settingsTemplate.gradle   # Repository configuration (Myket Maven)
│           ├── gradleTemplate.properties # Gradle JVM and build properties
│           └── proguard-user.txt         # Proguard / R8 keep rules
└── ProjectSettings/                      # Unity project settings
```

---

## Quick Start: Running the Sample

1. **Clone the repository:**
   ```bash
   git clone https://github.com/myket/myket-billing-unity-sample.git
   ```

2. **Open in Unity:**
   - Launch Unity Hub.
   - Click **Add** -> **Add project from disk** and select the repository root folder.
   - Open the project with your installed Unity version (e.g., Unity 6 or Unity 2022 LTS).

3. **Configure Package Name (Application ID):**
   - Go to **Edit > Project Settings > Player > Android tab**.
   - Under **Other Settings > Identification**, change **Package Name** to match the application registered on your Myket Developer Console (e.g., `com.yourcompany.yourapp`).

4. **Set Your Public Key and SKUs:**
   - Open `Assets/MyketIAB/Demo/IABTestUI.cs`.
   - Replace the `key` variable with your actual RSA Public Key from the Myket Developer Panel:
     ```csharp
     var key = "YOUR_MYKET_PUBLIC_KEY";
     MyketIAB.init(key);
     ```
   - Update the `skus` array with your registered product IDs:
     ```csharp
     string[] skus = {
         "your_product_sku_1",
         "your_product_sku_2",
         "your_subscription_sku"
     };
     ```

5. **Build and Run:**
   - Open the demo scene: `Assets/MyketIAB/Demo/IABTestScene.unity`.
   - Connect your Android test device via USB (with Developer Mode and USB Debugging enabled).
   - Go to **File > Build Settings...**, ensure the platform is set to **Android**, and click **Build and Run**.

---

## Integrating into Your Own Project

To add Myket In-App Billing to an existing Unity project:

### 1. Import Plugin Assets
Copy the `Assets/MyketIAB/` directory from this sample into your project's `Assets/` directory.

### 2. Configure Custom Gradle Templates
In Unity, go to **Project Settings > Player > Android tab > Publishing Settings**, and enable:
- **Custom Main Gradle Template** / **Custom Launcher Gradle Template**
- **Custom Gradle Settings Template**
- **Custom Proguard File**

#### A. Add Myket Maven Repository (`settingsTemplate.gradle`)
Add `https://maven.myket.ir` to the repository list in `Assets/Plugins/Android/settingsTemplate.gradle`:

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
    repositories {
        google()
        mavenCentral()
        maven { url 'https://maven.myket.ir' }
        flatDir {
            dirs "${project(':unityLibrary').projectDir}/libs"
        }
    }
}
```

#### B. Add Billing Dependency (`launcherTemplate.gradle` or `mainTemplate.gradle`)
Add the Myket Unity billing library under `dependencies` in `Assets/Plugins/Android/launcherTemplate.gradle`:

```groovy
dependencies {
    implementation project(':unityLibrary')
    implementation 'ir.myket:billing-unity:2.2'
}
```

Ensure Java 17 compatibility is set under the `android` block:

```groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    // ...
}
```

### 3. Configure ProGuard / R8 Rules
To prevent code obfuscation from breaking the native Android-to-Unity reflection bridge, ensure the following rules are present in `Assets/Plugins/Android/proguard-user.txt`:

```proguard
-keep class ir.myket.billingclient.** { *; }
-keep interface ir.myket.billingclient.** { *; }
-keep class com.android.vending.billing.** { *; }
-keep interface com.android.vending.billing.** { *; }
```

---

## Code Examples & API Guide

### Initialization
Initialize the billing client by passing your Base64-encoded RSA public key from the Myket Developer Portal:

```csharp
using MyketPlugin;

// Call early in your game lifecycle (e.g., Awake or Start)
string myketPublicKey = "YOUR_MYKET_PUBLIC_KEY";
MyketIAB.init(myketPublicKey);
```

### Event Handling
Subscribe to `IABEventManager` events to handle responses asynchronously. Always subscribe in `OnEnable` and unsubscribe in `OnDisable`:

```csharp
using System.Collections.Generic;
using UnityEngine;
using MyketPlugin;

public class BillingManager : MonoBehaviour
{
    void OnEnable()
    {
        IABEventManager.billingSupportedEvent += OnBillingSupported;
        IABEventManager.billingNotSupportedEvent += OnBillingNotSupported;
        IABEventManager.queryInventorySucceededEvent += OnQueryInventorySucceeded;
        IABEventManager.queryInventoryFailedEvent += OnQueryInventoryFailed;
        IABEventManager.purchaseSucceededEvent += OnPurchaseSucceeded;
        IABEventManager.purchaseFailedEvent += OnPurchaseFailed;
        IABEventManager.consumePurchaseSucceededEvent += OnConsumeSucceeded;
        IABEventManager.consumePurchaseFailedEvent += OnConsumeFailed;
    }

    void OnDisable()
    {
        IABEventManager.billingSupportedEvent -= OnBillingSupported;
        IABEventManager.billingNotSupportedEvent -= OnBillingNotSupported;
        IABEventManager.queryInventorySucceededEvent -= OnQueryInventorySucceeded;
        IABEventManager.queryInventoryFailedEvent -= OnQueryInventoryFailed;
        IABEventManager.purchaseSucceededEvent -= OnPurchaseSucceeded;
        IABEventManager.purchaseFailedEvent -= OnPurchaseFailed;
        IABEventManager.consumePurchaseSucceededEvent -= OnConsumeSucceeded;
        IABEventManager.consumePurchaseFailedEvent -= OnConsumeFailed;
    }

    void OnBillingSupported()
    {
        Debug.Log("Myket Billing is connected and ready.");
    }

    void OnBillingNotSupported(string error)
    {
        Debug.LogError("Myket Billing not supported: " + error);
    }

    void OnPurchaseSucceeded(MyketPurchase purchase)
    {
        Debug.Log($"Purchased product: {purchase.ProductId}, Token: {purchase.PurchaseToken}");
        
        // If consumable (e.g., coins, gems), consume it:
        MyketIAB.consumeProduct(purchase.ProductId);
    }

    void OnPurchaseFailed(string error)
    {
        Debug.LogError("Purchase failed: " + error);
    }

    void OnConsumeSucceeded(MyketPurchase purchase)
    {
        Debug.Log($"Product consumed successfully: {purchase.ProductId}");
        // Grant virtual currency or reward to the player
    }

    void OnConsumeFailed(string error)
    {
        Debug.LogError("Failed to consume product: " + error);
    }
}
```

### Querying SKUs and Purchases
Fetch product details (title, price, description) and check previously completed purchases:

```csharp
// Query SKU details along with user purchase state
string[] productIds = new string[] { "gem_pack_100", "monthly_pass" };
MyketIAB.queryInventory(productIds);

// Or query only SKU information
MyketIAB.querySkuDetails(productIds);

// Query only current user purchases
MyketIAB.queryPurchases();
```

### Purchasing Products
Initiate a purchase flow for a product:

```csharp
// Simple purchase
MyketIAB.purchaseProduct("gem_pack_100");

// Purchase with developer payload (optional security string for tracking)
MyketIAB.purchaseProduct("gem_pack_100", "unique_user_payload_123");
```

### Consuming Purchases
Consumable items (e.g., in-game coins, energy) must be consumed so the user can purchase them again:

```csharp
// Consume a single product by SKU
MyketIAB.consumeProduct("gem_pack_100");

// Consume multiple products at once
MyketIAB.consumeProducts(new string[] { "gem_pack_100", "gem_pack_200" });
```

### Subscriptions Support
Verify if the current device and installed Myket version support subscriptions:

```csharp
bool isSupported = MyketIAB.areSubscriptionsSupported();
Debug.Log("Subscriptions supported: " + isSupported);

// Purchase subscription with payload
MyketIAB.purchaseProduct("vip_monthly_subscription", "sub_payload");
```

---

## Debugging & Troubleshooting

### Enable Verbose Logging
Enable high-detail debug logs during development to trace communication between Unity and the Myket billing service:

```csharp
#if DEVELOPMENT_BUILD || UNITY_EDITOR
MyketIAB.enableLogging(true);
#endif
```

> [!WARNING]
> High-detail logs output sensitive billing and payload information to Android Logcat. Disable detailed logging before exporting production / release builds.

### Common Issues & Checklist

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| `billingNotSupportedEvent` fires | Myket application is not installed on device | Install the Myket app on the Android test device and log in. |
| "Item Unavailable" error | SKU not registered or application not published on portal | Verify that the SKU ID in code matches the developer portal and the APK package name is registered. |
| Signature verification failed | Incorrect Public Key | Ensure your RSA Public Key in `MyketIAB.init()` matches the key in the Myket Developer Panel under App Settings. |
| Crash or `ClassNotFoundException` in release build | ProGuard/R8 stripped native classes | Verify that rules from `Assets/Plugins/Android/proguard-user.txt` are included in your custom ProGuard file. |
| Gradle build failure: `ir.myket:billing-unity not found` | Missing Maven repository | Confirm `maven { url 'https://maven.myket.ir' }` is declared in `settingsTemplate.gradle`. |

---

## Support & Resources

- **Myket Developers Portal:** [developer.myket.ir](https://developer.myket.ir/)
- **In-App Billing Documentation:** [Myket Developer Docs](https://developer.myket.ir/docs/)
- **Issues & Contributions:** For plugin-related questions, open an issue in the repository or contact Myket developer support.
