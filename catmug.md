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

### Authentication
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
Create Users table and User object:
```c#
[DataTable(Name="Users")]
public class User
{
    public int Id { get; set; }
    
    public string ProviderId { get; set; }

    public string FirstName { get; set; }

    public string LastName { get; set; }
}

// ...snip...

private IMobileServiceTable<User> userTable = App.MobileService.GetTable<User>();

```

