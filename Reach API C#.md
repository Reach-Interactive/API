# SMS API Documentation
Our API gateway lets you connect to our interfaces over HTTP/S and allows your applications to integrate SMS over the internet. Our API uses a RESTful endpoint and our request and response payloads are formatted as JSON but we also provide GET as an alternative. 

**All you need to use our service is a free account and your authentication details are accessible after you sign up.**

### SMPP
We support SMPP Version 3.4. If you require an SMPP bind, please contact sales@reach-data.com and we will happy to set things up for you.

### Authentication
All requests require your username and password being passed across in the header. These can be found once you have logged in under Support > Developer > API Details

|Header   | Description                           |
|---------|---------------------------------------|
|username |	Your Reach Interactive API Username   |
|password |	Your Reach Interactive API Password   |

### HTTP Status Code

|Code |	Description                                           |
|-----|-------------------------------------------------------|
|200  |	Successful Request                                    |
|400  |	Details not correct or missing mandatory parameters   |
|401  |	Invalid Account details                               |
|402  |	Out of Credits                                        |
|403  |	Account forbidden for this action.                    |
|500  |	Service error.                                        |
|503  |	The Service is unavailable.                           |

### DLR Codes

DLR codes are the same for our API's and our SMPP binds.

|Code |	Description                   |
|-----|-------------------------------|
|000  |	Delivered                     |
|600  |	No credits to send            |
|601  |	No route                      |
|602  |	Blacklisted number detected   |
|603  |	Bad destination number        |
|604  |	Bad source number             |
|605  |	Target SMSC message queue     |
|606  |	Target SMSC submit fail       |
|607  |	General error                 |
|608  |	Spam message detected         |
|609  |	Validity period expired       |
|610  |	Unauthorised Source address   |
|611  |	Unknown DLR code              |
|612  |	Submit timeout                |

## Messaging

The URI for this section is

```
https://api.reach-interactive.com/sms/[Action]
```

### Balance Action

```
GET     /sms/balance
```

#### Required Parameters

There are no extra parameters for the Balance

```C#
using System;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
namespace Sms
{
    class Program
    {
        static void Main(string[] args)
        {
            var serializer = new JavaScriptSerializer();
            try
            {
                var url = "http://api.reach-interactive.com/sms/balance";
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.ContentType = "application/json";
                request.Method = "GET";
                request.Headers.Add("Username", "Your Reach Username");
                request.Headers.Add("Password", "Your Reach Password");
                using (var response = (HttpWebResponse)request.GetResponse())
                {
                    using (var reader = new StreamReader(response.GetResponseStream()))
                    {
                        var jsonResponse = reader.ReadToEnd();
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Balance: {1}, Description: {2}", 
                                            data["Success"], 
                                            data["Balance"], 
                                            data["Description"]);
                    }
                }
            }
            catch (WebException err)
            {
                using (var response = err.Response.GetResponseStream())
                using (var reader = new StreamReader(response))
                {
                    var jsonResponse = reader.ReadToEnd();
                    try
                    {
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}, HTTP Status Code: {2}", 
                                            data["Success"], 
                                            data["Description"],
                                            (int)((HttpWebResponse)err.Response).StatusCode);
                    }
                    catch (Exception)
                    {
                        Console.WriteLine(jsonResponse);
                    }
                }
            }
            catch (Exception err)
            {
                Console.WriteLine(err.Message);
            }
            Console.ReadKey();
        }
    }
}
```        
                
#### Returned Response

```json
[{
    "Success":true,
    "Balance":"xxx",
    "Description":"Credits Remaining"
}]
```      
        
### Message Send
Creates a new message object. We return an array of each Id generated. Per request, a max of 50 recipients can be entered.

```
POST    /sms/message
```
        
#### Required Parameters

|Parameter | Description                                                                                                                 |
|----------|-----------------------------------------------------------------------------------------------------------------------------|
|to        | The Number that you want to send to, can be multiple numbers seperated by a ',' or ';'                                      |
|from      | Who the message will appear to be from                                                                                      |
|message   | The message to send to the phone, the first 160 characters will be a single message, anything over will be split down into messages the size of 153 characters.<br> For Unicode, this is the text encoded in hexadecimal<br> For Binary messaging, this the User Data (140 octets max)|

#### Optional Parameters

|Parameter   |	Description                                                                                                                   | Default |
|------------|--------------------------------------------------------------------------------------------------------------------------------|---------|
|valid       |	How many hours that you require us to try to send the SMS for before it is expired, the minimum for this is 15 minutes (0.25) |	72      |
|reference   |	A reference that you want saved against the message in our system                                                             |         |
|callbackUrl |	The URL that you want us to send the delivery report to. See below in the Delivery Report for more information                |         |
|scheduled   |	The Date and Time you want the message to be sent (yyyy/MM/dd hh:mm)                                                          |         |
|coding      |	The type of message you would like to send, 1 = Text, 2 = Unicode, 3 = Binary, 4 = Auto	1                                     |         |
|udh         |	User Data Header                                                                                                              |         |

	
#### Example JSON Payload

```json
{
    "to" : "447xxxxxxxxx", 
    "from" : "Reach", 
    "message" : "Test Json Send" 
}
```         

```C#    
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
namespace Sms
{
    class Program
    {
        static void Main(string[] args)
        {
            var serializer = new JavaScriptSerializer();
            try
            {
                var url = "http://api.reach-interactive.com/sms/message";
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.ContentType = "application/json";
                request.Method = "POST";
                request.Headers.Add("Username", "Your Reach Username");
                request.Headers.Add("Password", "Your Reach Password");
                using (var streamWriter = new StreamWriter(request.GetRequestStream()))
                {
                    string json = "{ \"to\" : \"447xxxxxxxxx\", \"from\" : \"Reach\", \"message\" : \"Test API Message\"}";
                    streamWriter.Write(json);
                    streamWriter.Flush();
                    streamWriter.Close();
                }
                using (var response = (HttpWebResponse)request.GetResponse())
                {
                    using (var reader = new StreamReader(response.GetResponseStream()))
                    {
                        var jsonResponse = reader.ReadToEnd();
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        foreach (Dictionary<string, dynamic> item in data)
                        {
                            Console.WriteLine("Success: {0}, Id: {1}, Description: {2}", 
                                                item["Success"], 
                                                item["Id"], 
                                                item["Description"]);
                        }
                    }
                }
            }
            catch (WebException err)
            {
                using (var response = err.Response.GetResponseStream())
                using (var reader = new StreamReader(response))
                {
                    var jsonResponse = reader.ReadToEnd();
                    try
                    {
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}, HTTP Status Code: {2}",
                            data["Success"],
                            data["Description"],
                            (int) ((HttpWebResponse) err.Response).StatusCode);
                    }
                    catch (Exception)
                    {
                        Console.WriteLine(jsonResponse);
                    }
                }
            }
            catch (Exception err)
            {
                Console.WriteLine(err.Message);
            }
            Console.ReadKey();
        }
    }
}
```

#### Returned Response

```json
[{
    "Success":true,
    "Id":"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
    "Description":"Success"
}]
```

### Message Details

This can be used to retreive the details of a message sent over our API or SMPP binds.

```
GET     /sms/message/{Id}
```

```C#
using System;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
namespace Sms
{
    class Program
    {
        static void Main(string[] args)
        {
            var serializer = new JavaScriptSerializer();
            try
            {
                var url = "http://api.reach-interactive.com/sms/message/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa";
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.ContentType = "application/json";
                request.Method = "GET";
                request.Headers.Add("Username", "Your Reach Username");
                request.Headers.Add("Password", "Your Reach Password");
                using (var response = (HttpWebResponse)request.GetResponse())
                {
                    using (var reader = new StreamReader(response.GetResponseStream()))
                    {
                        var jsonResponse = reader.ReadToEnd();
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, To: {1}, From: {2}, Text: {3}, Sent: {4}, Status: {5}, Delivered: {6}, Code: {7}, Description: {8}",
                                            data[0]["Success"],
                                            data[0]["To"],
                                            data[0]["Originator"],
                                            data[0]["Text"],
                                            data[0]["Sent Date"],
                                            data[0]["Message Status"],
                                            data[0]["Delivered Date"],
                                            data[0]["DlrCode"],
                                            data[0]["Description"]);
                    }
                }
            }
            catch (WebException err)
            {
                using (var response = err.Response.GetResponseStream())
                using (var reader = new StreamReader(response))
                {
                    var jsonResponse = reader.ReadToEnd();
                    try
                    {
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}, HTTP Status Code: {2}", 
                                            data["Success"], 
                                            data["Description"],
                                            (int)((HttpWebResponse)err.Response).StatusCode);
                    }
                    catch (Exception)
                    {
                        Console.WriteLine(jsonResponse);
                    }
                }
            }
            catch (Exception err)
            {
                Console.WriteLine(err.Message);
            }
            Console.ReadKey();
        }
    }
}
```

#### Returned Response

```json
[{
    "Method":"H",
    "To":"447xxxxxxxxx",
    "Originator":"Reach",
    "Text":"Your Message",
    "Sent Date":"2016-06-23T15:27:08.497",
    "Message Status":"Delivered",
    "Delivered Date":"2016-06-23T15:27:17.973",
    "DlrCode":"000",
    "Description":"Unknown Error Code",
    "Reference":"",
    "Success":true
}]
```

### Message Delete

This can be used to delete a scheduled message that has not been sent.

```
DELETE  /sms/message/{Id}
```

```C#
using System;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
namespace Sms
{
    class Program
    {
        static void Main(string[] args)
        {
            var serializer = new JavaScriptSerializer();
            try
            {
                var url = "http://api.reach-interactive.com/sms/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa";
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.ContentType = "application/json";
                request.Method = "DELETE";
                request.Headers.Add("Username", "Your Reach Username");
                request.Headers.Add("Password", "Your Reach Password");
                using (var response = (HttpWebResponse)request.GetResponse())
                {
                    using (var reader = new StreamReader(response.GetResponseStream()))
                    {
                        var jsonResponse = reader.ReadToEnd();
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}", 
                                            data["Success"], 
                                            data["Description"]);
                    }
                }
            }
            catch (WebException err)
            {
                using (var response = err.Response.GetResponseStream())
                using (var reader = new StreamReader(response))
                {
                    var jsonResponse = reader.ReadToEnd();
                    try
                    {
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}, HTTP Status Code: {2}", 
                                            data["Success"], 
                                            data["Description"],
                                            (int)((HttpWebResponse)err.Response).StatusCode);
                    }
                    catch (Exception)
                    {
                        Console.WriteLine(jsonResponse);
                    }
                }
            }
            catch (Exception err)
            {
                Console.WriteLine(err.Message);
            }
            Console.ReadKey();
        }
    }
}
```

#### Returned Response

```json
{
    "Success":true,
    "Id":"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
    "Description":"Id Removed"
}
```

### Delivery Report

We can provide an HTTP GET response of the message
These are the parameters that will be passed back

|Parameter | Description                                         |
|----------|-----------------------------------------------------|
|msgid	   | The Id that was originally supplied in the API call |
|msisdn	   | The number that the delivery report is from         |
|timestamp | The date and time of the delivery report            |
|status    | The status of the delivery report                   |
|code      | The DLR code of the message                         |

#### Extra Information

If you require any more information passed back in the call back, you can add them to the URL that you supply.\
For example if you supply https://www.yourdomain.com/?CustomerId=123 the call back that then gets send back will be

```           
https://www.yourdomain.com/?CustomerId=123&MsgId=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa&MSISDN=447xxxxxxxxx&Timestamp=dd/MM/yyyy hh:mm:ss&Status=Delivered&Code=000
```

#### Status

|Status      | Description                                            |
|------------|--------------------------------------------------------|
|delivered   | Delivered successfully                                 |
|rejected    | Message rejected                                       |
|expired     | Message could not be delivered within valid time frame |
|undelivered | Message has not been delivered                         |

### Inbound Messages

After purchasing an Inbound long number or Short code keyword. Please Go to Inbox > Inbox in the menu to view / edit the settings for each Inbound

### HLR Action

```
GET     /sms/hlr/{msisdn}
```

```C#        
using System;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;
namespace Sms
{
    class Program
    {
        static void Main(string[] args)
        {
            var serializer = new JavaScriptSerializer();
            try
            {
                var url = "http://api.reach-interactive.com/sms/hlr/447xxxxxxxxx";
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                request.ContentType = "application/json";
                request.Method = "GET";
                request.Headers.Add("Username", "Your Reach Username");
                request.Headers.Add("Password", "Your Reach Password");
                using (var response = (HttpWebResponse)request.GetResponse())
                {
                    using (var reader = new StreamReader(response.GetResponseStream()))
                    {
                        var jsonResponse = reader.ReadToEnd();
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Msisdn: {1}, Mcc: {2}, Mnc: {3}, Description: {4}",
                                            data["Success"], 
                                            data["Msisdn"], 
                                            data["Mcc"], 
                                            data["Mnc"],
                                            data["Description"]);
                    }
                }
            }
            catch (WebException err)
            {
                using (var response = err.Response.GetResponseStream())
                using (var reader = new StreamReader(response))
                {
                    var jsonResponse = reader.ReadToEnd();
                    try
                    {
                        dynamic data = serializer.Deserialize<dynamic>(jsonResponse);
                        Console.WriteLine("Success: {0}, Description: {1}, HTTP Status Code: {2}", 
                                            data["Success"],
                                            data["Description"], 
                                            (int) ((HttpWebResponse) err.Response).StatusCode);
                    }
                    catch (Exception)
                    {
                        Console.WriteLine(jsonResponse);
                    }
                }
            }
            catch (Exception err)
            {
                Console.WriteLine(err.Message);
            }
            Console.ReadKey();
        }
    }
}
```

#### Returned Response

```json       
{
    "Success":true,
    "Msisdn":447912345678,
    "Mcc":234,
    "Mnc":15,
    "Description":"Active"
}
```
        
#### Descriptions

|Description | Description                                |
|------------|--------------------------------------------|
|active      | The number is registered and active        |
|absent      | The number is not registered on a network. |
|failed      | The number is not registered.              |
|unknown     | The number is unknown.                     |
