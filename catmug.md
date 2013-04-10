# Cincinnati All Things Mobile User Group
April 11, 2013

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
Fiddler: http://fiddler2.com

Windows Azure Mobile Services REST API Reference: http://msdn.microsoft.com/en-us/library/windowsazure/jj710108.aspx

SQL Azure: http://www.windowsazure.com/en-us/home/features/data-management/

### Logging In
Add some login buttons:
```xml
<Grid Grid.Row="0" Grid.ColumnSpan="2" Margin="0,0,0,20">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <StackPanel Height="74" Grid.Column="0">
            <TextBlock Foreground="#0094ff" FontFamily="Segoe UI Light" Margin="0,0,0,6">
            			<Run Text="WINDOWS AZURE MOBILE SERVICES"/>
            </TextBlock>
            <TextBlock Foreground="Gray" FontFamily="Segoe UI Light" FontSize="45" >
				<Run Text="catmug_demo"/>
            </TextBlock>
        </StackPanel>
        <StackPanel Grid.Column="1" HorizontalAlignment="Right" x:Name="LoginButtons" Visibility="Visible">
            <Button Click="Button_Click_1">Login Microsoft</Button>
            <Button Click="Button_Click_2">Login Google</Button>
        </StackPanel>
        <StackPanel Grid.Column="1" HorizontalAlignment="Right" x:Name="LoggedInPanel" Visibility="Collapsed">
            <Button Click="Button_Click_3">Logout</Button>
        </StackPanel>
    </Grid>
</Grid>
```
Login from C#:
```c#
MobileServiceUser user = await App.MobileService.LoginAsync(provider);
```
### Creating Users
Create Users table and User object:
```c#
[DataTable(Name="Users")]
public class User
{
    public int Id { get; set; }
    
    public string ProviderId { get; set; }
}

// ...snip...

private IMobileServiceTable<User> userTable = App.MobileService.GetTable<User>();
```

```c#
var results = await userTable.Where(u => u.ProviderId == user.UserId).ToListAsync();
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
            ProviderId = user.UserId
        };                

    await userTable.InsertAsync(existingUser);
}

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
http://blogs.msdn.com/b/carlosfigueira/archive/2012/10/25/getting-user-information-on-azure-mobile-services.aspx  
http://thejoyofcode.com/Fetching_a_basic_user_profile_in_Mobile_Services_Day_9_.aspx


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
