---
layout: post
title:  "Sync Adapter"
date:   2016-04-04 12:00:00
categories: Android
permalink: /archivers/syncadapter
comments: True
---

Without doubt, your app will communicate with the server and fetch some data for the the users to view, unless you have some extraordinary idea that is self sufficient within the app. Most likely, you will end up making frequent server calls to sync data. Here is where you will find the Sync adapter useful. For the 
Sync adapter you need an Account authenticator, which I have explained in my previous blog post. If you have not implemented an account authenticator, please do so. For someday when you end up sitting across Larry or Mark from you know where, you will thank me silently for the extra couple of billions that feature adds
to your deal. I take 100% commission in the form of retweets and likes.

However, if you do not deal with user accounts and still want to use the Sync adapter, this post will show you how to create one. I will assume you have not 
incorporated an account authenticator. So, I will show the stub implementations

**Why implement a Sync Adapter**

Well there are many ways to sync data with server calls, but when you have a tool called Sync adapter, that should be sufficient enough for you to use it ;-). Well if not, then consider this, Sync adapter usage is encouraged by Google, because it provides substantial savings in battery power. This is accomplished
by clubbing together many server calls. Whenever you make a server call, the device needs to wake up the radio (if not already awake). This causes some battery strain. The Sync adapter tries to reduce the battery strain by having the sync call performed at a time that is beneficial for most of the apps, so that radio is
not switched on and off again and for different calls from different apps. One call to rule them all. So let's get started. Create a android application with a MainActivity.java.

**Step 1: Create a stub Account authenticator, authenticator service**

This is what the previous tutorial was all about. If you followed that skip this step. Else, create the following two files:

* [Authenticator.java](https://github.com/sastagi/primesyncdemo/blob/master/app/src/main/java/com/primedroid/primesyncdemo/Authenticator.java)
* [AuthenticatorService.java](https://github.com/sastagi/primesyncdemo/blob/master/app/src/main/java/com/primedroid/primesyncdemo/AuthenticatorService.java)

Create the corresponding XML files and prefs files. Don't forget to add this info. in the manifest

**Step 2: Create a stub ContentProvider**

Now, for the sync adapter to run, you will need a content provider. That is a big topic in itself, so go ahead and create one, I will wait for you...still waiting. Naah, just kidding, you can create a stub provider too:

* [StubProvider.java](https://github.com/sastagi/primesyncdemo/blob/master/app/src/main/java/com/primedroid/primesyncdemo/StubProvider.java)

Make sure to add this to the manifest

**Step 3: Create a Sync Adapter**

This is the end of stubby stuff. This is where we create SyncAdapter.java. The main sync operation happens from onPerformSync which will be called from the MainActivity.java

```java
public void onPerformSync(
            Account account,
            Bundle extras,
            String authority,
            ContentProviderClient provider,
            SyncResult syncResult) {
    /*
     * Put the data transfer code here.
     */
    }
```
This is where you will perform the network operations, database calls. This happens on a separate thread, so you will not be blocking the UI thread. The following is a link to the file:

* [SyncAdapter.java](https://github.com/sastagi/primesyncdemo/blob/master/app/src/main/java/com/primedroid/primesyncdemo/SyncAdapter.java)

**Step 4: Create the Sync service to call the Sync Adapter**

The sync service is a service that will be running the sync adapter for you:

* [SyncService.java](https://github.com/sastagi/primesyncdemo/blob/master/app/src/main/java/com/primedroid/primesyncdemo/SyncService.java)

Make sure to add this to the manifest.

**Step 5: Call Sync Adapter from main**

First of write a method to create your account in the Sync adapter.

```java
public static Account CreateSyncAccount(Context context) {
        Account newAccount = new Account(
                ACCOUNT, ACCOUNT_TYPE);
        AccountManager accountManager =
                (AccountManager) context.getSystemService(
                        ACCOUNT_SERVICE);
        /*
         * Add the account and account type, no password or user data
         * If successful, return the Account object, otherwise report an error.
         */
        if (accountManager.addAccountExplicitly(newAccount, null, null)) {
            /*
             * If you don't set android:syncable="true" in
             * in your <provider> element in the manifest,
             * then call context.setIsSyncable(account, AUTHORITY, 1)
             * here.
             */
        } else {
            /*
             * The account exists or some other error occurred. Log this, report it,
             * or handle it internally.
             */
        }
        return newAccount;
    }
```    

Almost there. Now create the account and content resolver in onCreate and call the syncadapter as follows:

```java
	mAccount = CreateSyncAccount(this);

        mResolver = getContentResolver();
        
        Bundle syncSettingsBundle = new Bundle();
        syncSettingsBundle.putBoolean(ContentResolver.SYNC_EXTRAS_MANUAL,
                true);
        syncSettingsBundle.putBoolean(ContentResolver.SYNC_EXTRAS_EXPEDITED,
                true);
        ContentResolver.requestSync(mAccount, AUTHORITY, syncSettingsBundle);

        ContentResolver.setSyncAutomatically(mAccount, AUTHORITY, true);
        ContentResolver.addPeriodicSync(
                mAccount,
                AUTHORITY,
                Bundle.EMPTY,
                60L);
```    


Yes, you are done. That's it. Five steps and you have implemented your sync adapter. Feel free to download the sample app from:

[https://github.com/sastagi/primesyncdemo](https://github.com/sastagi/primesyncdemo)

The following are good links to learn more about sync adapter.

* [http://developer.android.com/training/sync-adapters/index.html](http://developer.android.com/training/sync-adapters/index.html)

* [http://blog.udinic.com/2013/07/24/write-your-own-android-sync-adapter/](http://blog.udinic.com/2013/07/24/write-your-own-android-sync-adapter/)

