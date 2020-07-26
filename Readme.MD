This repository helps to resolve the Workplace / Facebook integration problem with Microsoft Bto Framework.


1) Install the Facebook Adapter Nuget Package using the below script in the Package Manager Console

    	Install-Package Microsoft.Bot.Builder.Adapters.Facebook

2) Create an Adapter Class for Handling requests from FB / Worplace like the one below.

    	public class AdapterWithFBErrorHandler : FacebookAdapter
        	{
        	public AdapterWithFBErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
            	: base(configuration, logger)
        	{
            OnTurnError = async (turnContext, exception) =>
            {
                // Log any leaked exception from the application.
                logger.LogError(exception, $"[OnTurnError] unhandled error : {exception.Message}");

                // Send a message to the user
                await turnContext.SendActivityAsync("The bot encountered an error or bug.");
                await turnContext.SendActivityAsync("To continue to run this bot, please fix the bot source code.");

                // Send a trace activity, which will be displayed in the Bot Framework Emulator
                await turnContext.TraceActivityAsync("OnTurnError Trace", exception.Message, "https://www.botframework.com/schemas/error", "TurnError");
            };
        }
    }

3) Register the created Adapter class in Startup.cs file as below.

		// Create the FB Adapter for handling requests from FB / Workplace
		services.AddSingleton<FacebookAdapter, AdapterWithFBErrorHandler>();

4) Next step is to create an endpoint for handling requests from FB / Workplace
This can be achieved by creating a controller in BotController.cs file.

		[Route("api/facebook")]
    	[ApiController]
    	public class FBController : ControllerBase
    	{
        private readonly IBotFrameworkHttpAdapter Adapter;
        private readonly IBot Bot;

        public FBController(IBotFrameworkHttpAdapter adapter, IBot bot)
        {
            Adapter = adapter;
            Bot = bot;
        }

        [HttpPost, HttpGet]
        public async Task PostAsync()
        {
            // Delegate the processing of the HTTP POST to the adapter.
            // The adapter will invoke the bot.
            await Adapter.ProcessAsync(Request, Response, Bot);
        }
    }

5) To establish connection between Facebook and Azure, create a verify token and store in appsetting.json file like below.

	"FacebookVerifyToken": "", // You can create an unique token which needs to be configured in Facebook / Workplace
  	"FacebookAppSecret": "", // App Secret taken from FB when creating App in FB for Chatbot
  	"FacebookAccessToken": ""// Access token taken from FB when creating App in FB for Chatbot

 Happy Coding!!
