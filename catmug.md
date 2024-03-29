# Cincinnati All Things Mobile User Group  
April 11, 2013  
Brett Samblanet  
brettsam@microsoft.com  
@brett_sam

### Introduction to Windows Azure Mobile Services
Why?
* many mobile applications want to store some kind of state off of the device
* many mobile developers are experts on client development
* services can be tricky to set up and configure

Azure Mobile Services lets you create a backend for your mobile service in minutes with access to:
* data storage
* authentication with Facebook, Twitter, Google, or Microsoft accounts
* push notifications
* client SDKs for iOS, Android, Windows Phone 8, Windows 8 and HTML
* server-side scripting
* scale

Currently in preview. 10 free apps with each subscription.

Sign up for Azure: http://www.windowsazure.com/en-us/pricing/free-trial/
* 90-day free trial
* $0 spending limit

Azure mobile dev center: http://www.windowsazure.com/en-us/develop/mobile/

### Creating a Service and Running the QuickStart
All supported SDKs have quickstarts that can be found at http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started/

### Behind the Scenes: REST API and SQL Azure
http://www.thejoyofcode.com/Understanding_the_pipeline_and_sending_complex_objects_into_Mobile_Services_.aspx  
Fiddler: http://fiddler2.com  
Windows Azure Mobile Services REST API Reference: http://msdn.microsoft.com/en-us/library/windowsazure/jj710108.aspx  
SQL Azure: http://www.windowsazure.com/en-us/home/features/data-management/

### Logging In
http://blogs.msdn.com/b/carlosfigueira/archive/2012/10/23/troubleshooting-authentication-issues-in-azure-mobile-services.aspx  
Login from C#:
```c#
MobileServiceUser user = await App.MobileService.LoginAsync(provider);
```
### Creating Users
```c#
var results = await userTable.Where(u => u.ProviderId == App.MobileService.CurrentUser.UserId).ToListAsync();
User existingUser = results.FirstOrDefault();

if (existingUser == null)
{
    var dialog = new MessageDialog(
        String.Format("Welcome {0}! It looks like you're new. Would you like to create a new profile?",
        App.MobileService.CurrentUser.UserId));

    dialog.Commands.Add(new UICommand("Yes Please"));
    dialog.Commands.Add(new UICommand("No Thanks"));
    var result = await dialog.ShowAsync();

    if (result.Label != "Yes Please")
    {
        return;
    }

    existingUser = new User()
        {
            ProviderId = App.MobileService.CurrentUser.UserId
        };                

    await userTable.InsertAsync(existingUser);
}

this.LoggedInPanel.DataContext = existingUser;
RefreshTodoItems();
this.LoggedInPanel.Visibility = Visibility.Visible;
this.LoginButtons.Visibility = Visibility.Collapsed;
```
### Filtering Users on the Server
```javascript
function read(query, user, request) {

    query.where({
        ProviderId: user.userId
    });

    request.execute();
}
```
### User Identities
Azure Mobile Services Blogs:  
http://blogs.msdn.com/b/carlosfigueira/archive/2012/10/25/getting-user-information-on-azure-mobile-services.aspx  
http://thejoyofcode.com/Fetching_a_basic_user_profile_in_Mobile_Services_Day_9_.aspx  

OAuth documentation:  
https://developers.google.com/accounts/docs/OAuth2Login  
http://msdn.microsoft.com/en-us/library/live/hh243647.aspx  
http://msdn.microsoft.com/en-us/library/live/hh243646.aspx

Revoking access:  
https://www.google.com/dashboard/  
https://consent.live.com
```javascript
function read(query, user, request) {

    query.where({
        ProviderId: user.userId
    });

    request.execute({
        success: function(results) {

            if (results.length === 0) {
                var identities = user.getIdentities(),
                    url, fNameId, lNameId;

                if (identities.google) {
                    var googleAccessToken = identities.google.accessToken;
                    url = 'https://www.googleapis.com/oauth2/v1/userinfo?access_token=' + googleAccessToken;
                    fNameId = 'given_name';
                    lNameId = 'family_name';
                } else if (identities.microsoft) {
                    var liveAccessToken = identities.microsoft.accessToken;
                    url = 'https://apis.live.net/v5.0/me/?method=GET&access_token=' + liveAccessToken;
                    fNameId = 'first_name';
                    lNameId = 'last_name';
                }

                var requestCallback = function(err, resp, body) {
                        var userData = JSON.parse(body);
                        var newUser = {};
                        newUser.FirstName = userData[fNameId];
                        newUser.LastName = userData[lNameId];
                        results.push(newUser);
                        request.respond(statusCodes.OK, results);
                    };

                var req = require('request');
                var reqOptions = {
                    uri: url,
                    headers: {
                        Accept: "application/json"
                    }
                };
                req(reqOptions, requestCallback);

            } else {
                request.respond(statusCodes.OK, results);
            }
        }
    });
}
```
### Associating Users with TodoItems
```c#
private void ButtonSave_Click(object sender, RoutedEventArgs e)
{
    var todoItem = new TodoItem { Text = TextInput.Text, CreatedBy = App.MobileService.CurrentUser.UserId };
    InsertTodoItem(todoItem);
}
```
### Using the mssql Object to Perform a Join
```javascript
function read(query, user, request) {

    var sql = 'SELECT t.id, t.text, t.complete, t.CreatedBy, \
               u.FirstName + \' \' + u.LastName AS CreatedByFullName \
               FROM TodoItem t \
               JOIN Users u ON CreatedBy=ProviderId \
               WHERE t.complete=0 \
               ORDER BY t.id';

    mssql.query(sql, {
        success: function(results){
            console.log(results);
            request.respond(statusCodes.OK, results);
        }
    });
}
```
### Registering Push Tokens
http://www.windowsazure.com/en-us/develop/mobile/resources/#header-3  
http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Windows-Store-app-Add-Push-Notifications-to-your-apps-with-Windows-Azure-Mobile-Services
```c#
PushNotificationChannel CurrentChannel;

private async void RegisterPushChannel()
{
    CurrentChannel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    IMobileServiceTable<DeviceToken> deviceTokenTable = App.MobileService.GetTable<DeviceToken>();
    var channel = new DeviceToken { Token = CurrentChannel.Uri, UserId = App.MobileService.CurrentUser.UserId };
    await deviceTokenTable.InsertAsync(channel);
}
```
```javascript
function insert(item, user, request) {
    tables.getTable('DeviceTokens').where({
        Token: item.Token,
        UserId: user.userId
    }).read({
        success: insertChannelIfNotFound
    });

    function insertChannelIfNotFound(existingChannels) {
        if (existingChannels.length > 0) {
            request.respond(statusCodes.OK, existingChannels[0]);
        } else {
            request.execute();
        }
    }
}
```
### Sending a Push when a TodoItem is Completed
```javascript
function update(item, user, request) {

    delete item.CreatedByFullName;

    request.execute();

    if (item.CreatedBy === user.userId) {
        console.log('Same user. No push to send.');
        return;
    }

    tables.getTable('DeviceTokens').where({
        UserId: item.CreatedBy
    }).read({
        success: getUserAndSendPush
    });

    function getUserAndSendPush(deviceTokens) {
        tables.getTable('Users').where({
            ProviderId: user.userId
        }).read({
            success: function(userResults) {
                deviceTokens.forEach(function(deviceToken) {
                    push.wns.sendToastText01(deviceToken.Token, {
                        text1: userResults[0].FirstName + ' completed task: ' + item.text
                    });
                });
            }
        });
    }
}
```
### Blob Storage
http://www.nickharris.net/2012/11/how-to-upload-an-image-to-windows-azure-storage-using-mobile-services/

### Scheduler
http://hashtagfail.com/post/38488024433/mobile-services-scheduler  
http://thejoyofcode.com/Periodic_Notifications_with_Windows_Azure_Mobile_Services.aspx

### Command Line Interface
http://thejoyofcode.com/Getting_started_with_the_CLI_and_backing_up_your_scripts_Day_4_.aspx  
http://thejoyofcode.com/More_CLI_ndash_changing_your_Mobile_Services_workflow_Day_5_.aspx  
http://www.windowsazure.com/en-us/develop/mobile/tutorials/command-line-administration/

### Client Resources
#### iOS
http://www.windowsazure.com/en-us/develop/mobile/ios/  
http://thejoyofcode.com/

#### Android
http://chrisrisner.com/Announcing-Android-Support-for-Mobile-Services  
http://hashtagfail.com/post/44606054459/mobile-services-android-typed-untyped

#### HTML
http://channel9.msdn.com/Series/Windows-Azure-Mobile-Services/Getting-Started-with-the-Mobile-Services-HTML-Client  
http://blog.stevensanderson.com/2013/03/13/touralot-an-ios-app-built-with-phonegap-knockout-and-azure-mobile-services/
