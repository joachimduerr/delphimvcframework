#DMVCFramework features
  * Simple to use, check the ["Getting Started: 5 minutes guide"](https://danieleteti.gitbooks.io/delphimvcframework/content/chapter_getting_started.html) and you will be up and running in 5 minutes or less!
  * RESTful (RMM Level 3) compliant
  * Stable and solid, used by small/mid/big projects since 2010
  * Support group at https://www.facebook.com/groups/delphimvcframework with more than 700 active members
  * Can be used in load balanced environment using Redis (http://Redis.io) [dev]
  * Can be used in load balanced environment using MySQL [user contrib]
  * Wizard for the Delphi IDE. It makes DelphiMVCFramework even more simple to use!
  * Optional session support
  * JSON Web Token Support (JWT)
  * Extendable using middlewares (simple hooks to handle request/response)
  * CORS support
  * Basic Authentication
  * Controllers inheritance
  * Fancy URL with parameter mappings
  * Specialied renders to generate text, HTML, JSON
  * Powerful mapper to map json to objects and datasets to objects
  * Can be packaged as stand alone server, apache module (XE6 or better) and ISAPI dll
  * Integrated RESTClient
  * Works with XE3, XE4, XE5, XE6, XE7, XE8, Delphi 10 Seattle, Delphi 10.1 Berlin
  * Completely unit tested
  * There is a sample for each functionality
  * There is a complete set of trainings about it, but the samples are included in the project
  * Server side generated pages using Mustache (https://mustache.github.io/) for Delphi (https://github.com/synopse/dmustache)
  * Specific trainings are available (ask me for a date and a place)
  * Messaging extension using STOMP (beta)
  * Automatic documentation through /system/describeserver.info
  * Driven by its huge community (Facebook group https://www.facebook.com/groups/delphimvcframework)
  * Simple and [documented](https://github.com/danieleteti/delphimvcframework/blob/master/docs/ITDevCON%202013%20-%20Introduction%20to%20DelphiMVCFramework.pdf)
  * Check the [DMVCFramework Developer Guide](https://danieleteti.gitbooks.io/delphimvcframework/content/) (work in progress)
  
## Trainings, consultancy or custom development service
As you know, good support on open source software is a must for professional users.
If you need trainings, consultancy or custom developments on DelphiMVCFramework, send an email to *dmvcframework at bittime dot it*. Alternatively you can send a request using the [contacts forms](http://www.bittimeprofessionals.it/contatti) on [bittimeprofessionals website](http://www.bittimeprofessionals.it). bit Time Professionals is the company behind DelphiMVCFramework, al the main developers works there.


## Sub Projects
DelphiMVCFramework contains also a lot of indipendent code that can be used in other kind of project. 

These are the most notable:

  * Mapper (convert JSON in Object and back, ObjectList in JSONArray and back, DataSets in JSONArray or ObjectList and back)
  * DelphiRedisClient (https://github.com/danieleteti/delphiredisclient)

##Samples and documentation
DMVCFramework is provided with a lot of examples focused on specific functionality.
All samples are in [Samples](https://github.com/danieleteti/delphimvcframework/tree/master/samples) folder.
Check the [DMVCFramework Developer Guide](https://danieleteti.gitbooks.io/delphimvcframework/content/) (work in progress).


#Sample Server
Below a basic sample of a DMVCFramework server wich can be deployed as standa-alone application, as an Apache module or as ISAPI dll. This flexibility is provided by the Delphi WebBroker framework (built-in in Delphi since Delphi 4).

To create this server, you have to create a new Delphi Projects -> WebBroker -> WebServerApplication. Then add the following changes to the webmodule.
```delphi
unit WebModuleUnit1;

interface

uses System.SysUtils, System.Classes, Web.HTTPApp, MVCFramework {this unit contains TMVCEngine class};

type
  TWebModule1 = class(TWebModule)
    procedure WebModuleCreate(Sender: TObject);
    procedure WebModuleDestroy(Sender: TObject);

  private
    MVC: TMVCEngine;

  public
    { Public declarations }
  end;

var
  WebModuleClass: TComponentClass = TWebModule1;

implementation

{$R *.dfm}

uses UsersControllerU; //this is the unit where is defined the controller

procedure TWebModule1.WebModuleCreate(Sender: TObject);
begin
  MVC := TMVCEngine.Create(Self);
  MVC.Config['document_root'] := 'public_html'; //if you need some static html, javascript, etc (optional)
  MVC.AddController(TUsersController); //see next section to know how to create a controller
end;

procedure TWebModule1.WebModuleDestroy(Sender: TObject);
begin
  MVC.Free;
end;

end.
```

Remember that the files inside the redist folder *must* be in the executable path or in the system path. If starting the server whithin the IDE doesn't works, try to run the executable outside the IDE and check the dependencies.
That's it! You have just created your first DelphiMVCFramework. Now you have to add a controller to respond to the http request.

#Sample Controller
Below a basic sample of a DMVCFramework controller with 2 action

```delphi
unit UsersControllerU;
  
interface
  
uses 
  MVCFramework;
 
type 
   [MVCPath('/users')]
   TUsersController = class(TMVCController)
   public
    
    //The following action will be with a GET request like the following
    //http://myserver.com/users/3
    [MVCPath('/($id)')]
    [MVCProduces('application/json')]
    [MVCHTTPMethod([httpGET])]
    [MVCDoc('Returns the users list as a JSON Array of JSON Objects')]
    procedure GetUsers(CTX: TWebContext);

    //The following action will be with a PUT request like the following
    //http://myserver.com/users/3
    //and in the request body there should be a serialized TUser
    [MVCPath('/($id)')]
    [MVCProduce('application/json')]
    [MVCHTTPMethod([httpPUT])]
    [MVCDoc('Update a user')]    
    procedure UpdateUser(CTX: TWebContext);

    //The following action will respond to a POST request like the following
    //http://myserver.com/users
    //and in the request body there should be the new user to create as json
    [MVCPath]
    [MVCProduce('application/json')]
    [MVCHTTPMethod([httpPOST])]
    [MVCDoc('Create a new user, returns the id of the new user')]
    procedure CreateUser(CTX: TWebContext);

  end;
 
implementation

uses
  MyTransactionScript; //contains actual data access code
  
{ TUsersController }
 
procedure TUsersController.GetUsers(CTX: TWebContext);
var
  User: TUser;
begin
  User := GetUserById(CTX.Request.Parameters['id'].ToInteger);
  Render(User);
end;

procedure TUsersController.UpdateUser(CTX: TWebContext);
var
  User: TUser;
begin
  User := CTX.Request.BodyAs<TUser>;
  SaveUser(User);
  Render(User);
end;	
  
procedure TUsersController.CreateUser(CTX: TWebContext);
var
  User: TUser;
begin
  User := CTX.Request.BodyAs<TUser>;
  CreateUser(User);
  Render(User);
end;	
  
end.
```

Now you have a performant RESTful server wich respond to the following URLs:
- GET /users/($id)		(eg. /users/1, /users/45 etc)
- PUT /users/($id)		(eg. /users/1, /users/45 etc with the JSON data in the request body)
- POST /users			(the JSON data must be in the request body)

###Quick Creation of DelphiMVCFramework Server

If you dont plan to deploy your DMVCFramework server behind a webserver (apache or IIS) you can also pack more than one listener application server into one single executable. In this case, the process is a bit different and involves the creation of a listener context. However, create a new server is a simple task:

```delphi
uses
  MVCFramework.Server,
  MVCFramework.Server.Impl;

var
  LServerListener: IMVCListener;
begin
  LServerListener := TMVCListener.Create(TMVCListenerProperties.New
	 .SetName('Listener1')
	 .SetPort(5000)
	 .SetMaxConnections(1024)
	 .SetWebModuleClass(YourServerWebModuleClass)
   );  

  LServerListener.Start;
  LServerListener.Stop;
end;
```

If you want to add a layer of security (in its WebModule you should add the security middleware):

```delphi
uses
  MVCFramework.Server,
  MVCFramework.Server.Impl,
  MVCFramework.Middleware.Authentication;

procedure TTestWebModule.WebModuleCreate(Sender: TObject);
begin
  FMVCEngine := TMVCEngine.Create(Self);
	
  // Add Yours Controllers
  FMVCEngine.AddController(TYourController);
	
  // Add Security Middleware
  FMVCEngine.AddMiddleware(TMVCBasicAuthenticationMiddleware.Create(
    TMVCDefaultAuthenticationHandler.New
    .SetOnAuthentication(
		procedure(const AUserName, APassword: string;
		  AUserRoles: TList<string>; var IsValid: Boolean; 
		  const ASessionData: TDictionary<String, String>)
		begin
		  IsValid := AUserName.Equals('dmvc') and APassword.Equals('123');
		end
		)
    ));
end;  
```

In stand alone mode you can work with a context that supports multiple listeners servers:

```delphi
uses
  MVCFramework.Server,
  MVCFramework.Server.Impl;

var
  LServerListenerCtx: IMVCListenersContext;

begin
  LServerListenerCtx := TMVCListenersContext.Create;

  LServerListenerCtx.Add(TMVCListenerProperties.New
    .SetName('Listener1')
    .SetPort(6000)
    .SetMaxConnections(1024)
    .SetWebModuleClass(WebModuleClass1)
    );

  LServerListenerCtx.Add(TMVCListenerProperties.New
    .SetName('Listener2')
    .SetPort(7000)
    .SetMaxConnections(1024)
    .SetWebModuleClass(WebModuleClass2)
    );

  LServerListenerCtx.StartAll;
end;  
```

###Links
Feel free to ask questions on the "Delphi MVC Framework" facebook group (https://www.facebook.com/groups/delphimvcframework).

http://www.danieleteti.it/2013/04/18/sneak-peek-to-simple-integration-between-dmvcframework-and-dorm/
