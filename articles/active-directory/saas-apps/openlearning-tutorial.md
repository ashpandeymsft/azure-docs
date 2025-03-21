---
title: 'Tutorial: Azure AD SSO integration with OpenLearning'
description: Learn how to configure single sign-on between Azure Active Directory and OpenLearning.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/31/2022
ms.author: jeedes

---

# Tutorial: Azure AD SSO integration with OpenLearning

In this tutorial, you'll learn how to integrate OpenLearning with Azure Active Directory (Azure AD). When you integrate OpenLearning with Azure AD, you can:

* Control in Azure AD who has access to OpenLearning.
* Enable your users to be automatically signed-in to OpenLearning with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* OpenLearning single sign-on (SSO) enabled subscription.

> [!NOTE]
> This integration is also available to use from Azure AD US Government Cloud environment. You can find this application in the Azure AD US Government Cloud Application Gallery and configure it in the same way as you do from public cloud.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* OpenLearning supports **SP** initiated SSO.

## Add OpenLearning from the gallery

To configure the integration of OpenLearning into Azure AD, you need to add OpenLearning from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **OpenLearning** in the search box.
1. Select **OpenLearning** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

 Alternatively, you can also use the [Enterprise App Configuration Wizard](https://portal.office.com/AdminPortal/home?Q=Docs#/azureadappintegration). In this wizard, you can add an application to your tenant, add users/groups to the app, assign roles, as well as walk through the SSO configuration as well. [Learn more about Microsoft 365 wizards.](/microsoft-365/admin/misc/azure-ad-setup-guides)

## Configure and test Azure AD SSO for OpenLearning

Configure and test Azure AD SSO with OpenLearning using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in OpenLearning.

To configure and test Azure AD SSO with OpenLearning, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure OpenLearning SSO](#configure-openlearning-sso)** - to configure the single sign-on settings on application side.
    1. **[Create OpenLearning test user](#create-openlearning-test-user)** - to have a counterpart of B.Simon in OpenLearning that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **OpenLearning** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

   ![Screenshot shows to edit Basic S A M L Configuration.](common/edit-urls.png "Basic Configuration")

1. On the **Basic SAML Configuration** section, if you have **Service Provider metadata file**, perform the following steps:

	a. Click **Upload metadata file**.

    ![Screenshot shows to upload metadata file.](common/upload-metadata.png "Metadata")

	b. Click on **folder logo** to select the metadata file and click **Upload**.

	![Screenshot shows to choose metadata file.](common/browse-upload-metadata.png "Folder")

	c. After the metadata file is successfully uploaded, the **Identifier** value gets auto populated in Basic SAML Configuration section.

	d. In the **Sign-on URL** text box, type a URL using the following pattern:
    `https://www.openlearning.com/saml-redirect/<institution_id>/<idp_name>/`

	> [!Note]
	> If the **Identifier** value does not get auto populated, then please fill in the value manually according to your requirement. The Sign-on URL value is not real. Update this value with the actual Sign-on URL. Contact [OpenLearning Client support team](mailto:dev@openlearning.com) to get this value. You can also refer to the patterns shown in the **Basic SAML Configuration** section in the Azure portal.

1. OpenLearning Identity Authentication application expects the SAML assertions in a specific format, which requires you to add custom attribute mappings to your SAML token attributes configuration. The following screenshot shows the list of default attributes.

	![image](common/default-attributes.png)

1. In addition to above, OpenLearning Identity Authentication application expects few more attributes to be passed back in SAML response which are shown below. These attributes are also pre populated but you can review them as per your requirements.

    | Name | Source Attribute|
	| ---------------| --------------- |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:2.16.840.1.113730.3.1.241 | user.displayname |
    | urn:oid:1.3.6.1.4.1.5923.1.1.1.9 | user.extensionattribute1 |
    | urn:oid:1.3.6.1.4.1.5923.1.1.1.6 | user.objectid |
    | urn:oid:2.5.4.10 | user.companyname |

1. On the **Set up single sign-on with SAML** page, in the **SAML Signing Certificate** section,  find **Certificate (Base64)** and select **Download** to download the certificate and save it on your computer.

	![Screenshot shows the Certificate download link.](common/certificatebase64.png "Certificate")

1. On the **Set up OpenLearning** section, copy the appropriate URL(s) based on your requirement.

	![Screenshot shows to copy Configuration appropriate U R L.](common/copy-configuration-urls.png "Configuration")
	
1. OpenLearning application expects to enable token encryption in order to make SSO work. To activate token encryption, go to the **Azure Active Directory** > **Enterprise applications** and select **Token encryption**. For more information, please refer this [link](../manage-apps/howto-saml-token-encryption.md).

    ![Screenshot shows the activation of Token Encryption.](./media/openlearning-tutorial/token.png "Token Encryption")

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to OpenLearning.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **OpenLearning**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure OpenLearning SSO

1. Log in to your OpenLearning company site as an administrator.

1. Go to **SETTINGS** > **Integrations** and click **ADD** under SAML Identity Provider(IDP) Configuration.

1. In the **SAML Identity Provider** page, perform the following steps:

    ![Screenshot shows SAML settings](./media/openlearning-tutorial/certificate.png "SAML settings")

    1. In the **Name (required)** textbox, type a short configuration name.

    1. Copy **Reply(ACS) Url** value, paste this value into the **Reply URL** text box in the **Basic SAML Configuration** section in the Azure portal.

    1. In the **Entity ID/Issuer URL (required)** textbox, paste the **Azure AD Identifier** value which you have copied from the Azure portal.

    1. In the **Sign-In URL (required)** textbox, paste the **Login URL** value which you have copied from the Azure portal.

    1. Open the downloaded **Certificate (Base64)** from the Azure portal into Notepad and paste the content into the **Certificate (required)** textbox.

    1. Download the **Metadata XML** into Notepad and upload the file into **Basic SAML Configuration** section in the Azure portal.

    1. Click **Save**.

### Create OpenLearning test user

In this section, a user called Britta Simon is created in OpenLearning. OpenLearning supports just-in-time user provisioning, which is enabled by default. There is no action item for you in this section. If a user doesn't already exist in OpenLearning, a new one is created after authentication.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration with following options. 

* Click on **Test this application** in Azure portal. This will redirect to OpenLearning Sign-on URL where you can initiate the login flow. 

* Go to OpenLearning Sign-on URL directly and initiate the login flow from there.

* You can use Microsoft My Apps. When you click the OpenLearning tile in the My Apps, this will redirect to OpenLearning Sign-on URL. For more information about the My Apps, see [Introduction to the My Apps](../user-help/my-apps-portal-end-user-access.md).

## Next steps

Once you configure OpenLearning you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).