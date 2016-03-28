---
layout: post
title:  "Account authenticator tutorial"
date:   2016-03-27 12:00:00
categories: Android
permalink: /archivers/accountauthenticator
comments: True
---

Let us start at the beginning. You have an android app and want to allow users to create an account on your app. You might think about storing the user identifying token in shared preferences or sqllite DB, or just make a server call every time. There  may be some solid implementations for these, but Android actually provides a mechanism for account creation and management. So let's make use of it. But how? This is what grinds my gears. 
The documentation is sparse in my opinion and if not for some good blogs (mentioned at the bottom) I'd be stuck too. So this is one more attempt at demystifying the Account Authenticator. 

The account authenticator route involves a lot of boiler plate code. But this is because as your app becomes popular, you will need to cover for all the edge cases which are thoroughly researched and provided for in the account authenticator. You don't have to implement all of them, but address them on a case by case basis. Also, if you plan to use the Sync adapter (highly recommend for server and local device data syncs), you will need an account created using Account Authenticator. 
So let's get going.

**Objective**

We will create a sample app where the user will be presented with a login screen on launch. The user can then login and every time time the user launches the app after that, the user will see the logged in screen. So create an android app with a single activity (MainActivity.java).

**Step 1: Create a class that extends AbstractAccountAuthenticator**

Create a new class called [AccountAuthenticator](https://github.com/sastagi/primeaccount/blob/master/app/src/main/java/com/primedroid/primeaccount/AccountAuthenticator.java) and make it extend [AbstractAccountAuthenticator](http://developer.android.com/reference/android/accounts/AbstractAccountAuthenticator.html). You will need to implement seven methods. Stub them to return null for now.

**Step 2: Create an activity with the login screen**

This is the activity which the user will be directed to, if the Account authenticator is unable to find a token. In the sample app this is called [LoginActivity](https://github.com/sastagi/primeaccount/blob/master/app/src/main/java/com/primedroid/primeaccount/LoginActivity.java). Provide a couple of Edittext fields and a button for the user to login.
 
**Step 3: Create a java file to store all account related info**

We will call this [AccountInfo](https://github.com/sastagi/primeaccount/blob/master/app/src/main/java/com/primedroid/primeaccount/AccountInfo.java) and store the following string constants: `ACCOUNT_TYPE`, `ACCOUNT_NAME` and `AUTHTOKENTYPE`

**Step 4: Create the service to bind the AccountAuthenticator**

We will need a service to bind the AccountAuthenticator with the activity. We will call this [AccountAuthenticatorService](https://github.com/sastagi/primeaccount/blob/master/app/src/main/java/com/primedroid/primeaccount/AccountAuthenticatorService.java). Don't forget to include the information for this service in the manifest.

**Step 5: Add the required permissions to the manifest**

We will need the following four permissions to make this work: `GET_ACCOUNTS`, `USE_CREDENTIALS`, `MANAGE_ACCOUNTS`, `AUTHENTICATE_ACCOUNTS`

**Step 6: Create the xml files for settings screen**

One of the cool features of using the account authenticator is that your app pops up in the accounts screen under the Android settings section. We need to provide the information for the representation of this using xml. So create the following two files and put them under resources/xml: [authenticator.xml](https://github.com/sastagi/primeaccount/blob/master/app/src/main/res/xml/authenticator.xml), [prefs.xml](https://github.com/sastagi/primeaccount/blob/master/app/src/main/res/xml/prefs.xml)

`CAUTION` - Make sure that the accountType attribute value in authenticator.xml matches the string in ACCOUNT_TYPE.

**Step 7: Check for token in MainActivity on launch**

This is where the fun begins. Now you will need an instance of AccountManager here. So create one (mAccountManager). Create a method checkAccountExists and pass authTokenType (this is the token type defined under AccountInfo) to it.

```java
private void checkAccount(final String authTokenType) {
        final Account availableAccounts[] = mAccountManager.getAccountsByType(AccountInfo.ACCOUNT_TYPE);

        if (availableAccounts.length == 0) {
            addNewAccount(AccountInfo.ACCOUNT_TYPE,authTokenType);
        } else {
            String name[] = new String[availableAccounts.length];
            for (int i = 0; i < availableAccounts.length; i++) {
                name[i] = availableAccounts[i].name;
            }
            getExistingAccountAuthToken(availableAccounts[0], authTokenType);
        }
    }
```

In this method we are asking the AccountManager to return all the accounts matching this account type. If there are no accounts found, we will call addNewAccount, which directs the user to the login screen. We will talk about the getExistingAccountAuthToken method in step 10.


```java
private void addNewAccount(String accountType, String authTokenType) {
        final AccountManagerFuture<Bundle> future = mAccountManager.addAccount(accountType, authTokenType, null, null, this, new AccountManagerCallback<Bundle>() {
            @Override
            public void run(AccountManagerFuture<Bundle> future) {
                try {
                    Bundle bnd = future.getResult();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, null);
    }
```

The addNewAccount basically calls addAccount which needs to be implemented in AccountAuthenticator.

**Step 8: Implement addAccount method of AbstractAccountAuthenticator**

The following method calls the LoginActivity:

```java
@Override
    public Bundle addAccount(AccountAuthenticatorResponse response, String accountType, String authTokenType, String[] requiredFeatures, Bundle options) throws NetworkErrorException {
        final Intent intent = new Intent(MainActivity.mContext,  LoginActivity.class);
        intent.putExtra(AccountAuthenticator.ARG_ACCOUNT_TYPE, accountType);
        intent.putExtra(AccountAuthenticator.ARG_AUTH_TYPE, authTokenType);
        intent.putExtra(AccountAuthenticator.ARG_IS_ADDING_NEW_ACCOUNT, true);
        intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);

        final Bundle bundle = new Bundle();
        bundle.putParcelable(AccountManager.KEY_INTENT, intent);
        return bundle;
    }
```    

**BREAK**

If you have come so far, I admire your patience. So at this point, try to run your app. You should be directed to the Login activity (since you do not have an account with account manager). If not, watch out for these common gotchas:

* Check name of account type in authenticator.xml
* Check permissions in manifest
* Check declarations for activities and services

**Step 9: Add account to account manager**

When the user clicks the login button,  we want to add the account to the Account manager. So create a method called addAccount in your LoginActivity and call it on the click of login button:

```java
protected void addAccount(){
        //This is where you make the server call and get the authorization tokens. Adding dummy stuff here.
        String username = mEmailField.getText().toString();
        String password = mPasswordField.getText().toString();
        String token = UUID.randomUUID()+"-prime";

        if(TextUtils.isEmpty(username)||TextUtils.isEmpty(password)){
            mInvalidCredentials.setVisibility(View.GONE);
        } else {
            final Account account = new Account(username, AccountInfo.ACCOUNT_TYPE);
            MainActivity.mAccountManager.addAccountExplicitly(account, password, null);
            MainActivity.mAccountManager.setAuthToken(account, AccountInfo.AUTHTOKENTYPE, token);
            mInvalidCredentials.setVisibility(View.GONE);
            startActivity(new Intent(this, MainActivity.class));
            finish();
        }
    }
```

If you run the app now and login using any non-empty credentials, you will be taken to the main activity. Subsequent launches will take you straight to the main activity, since you have an account with the Account manager.

**Step 10: Get auth token for existing account**

Addressing step 7 here. When the user launches the app again, getExistingAccountAuthToken will be called returning information regarding the account. You can get the username, auth token and any other information that you store during account creation from here.

```java
private void getExistingAccountAuthToken(Account account, String authTokenType) {
        final AccountManagerFuture<Bundle> future = mAccountManager.getAuthToken(account, authTokenType, null, this, null, null);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Bundle bundle = future.getResult();
                    final String authtoken = bundle.getString(AccountManager.KEY_ACCOUNT_NAME);
                    mUsername.setText(authtoken);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

**Step 11: DONE!!!**

Yes, you are done. That's it. Ten steps and you have implemented your account authenticator. Feel free to download the sample app from:

[https://github.com/sastagi/primeaccount](https://github.com/sastagi/primeaccount)

The account authenticator has many more features than mentioned here. They cover most of the edge cases that you are bound to encounter as you app becomes popular. Some other interesting blogs on Account authenticator:

* [http://blog.udinic.com/2013/04/24/write-your-own-android-authenticator](http://blog.udinic.com/2013/04/24/write-your-own-android-authenticator)

* [https://www.finalconcept.com.au/article/view/android-account-manager-step-by-step-2](https://www.finalconcept.com.au/article/view/android-account-manager-step-by-step-2)

* [http://www.c99.org/2010/01/23/writing-an-android-sync-provider-part-1/](http://www.c99.org/2010/01/23/writing-an-android-sync-provider-part-1/)