# Paso 5 - Implementando llamadas a Azure OpenAI con las Preguntas del Chat

1. Modificar el archivo ubicado en /Components/Pages/Home.razor, con el siguiente contenido:
```
@using Azure;
@using Azure.AI.OpenAI;
@using System.Text.Json;
@using static System.Environment;
@page "/"
@rendermode InteractiveServer
<PageTitle>Chat GPT - Blazor WASM - TCD 2024</PageTitle>
<div class="h1 mb-4 text-center"> Bienvenido a Chat GPT desarrollado con Blazor Web Assembly </div>
<div style="line-height:40px;" class="h4 mb-4 text-center"> Aquí puedes buscar realizar una pregunta sobre el Tech
  Community Day 2024, horarios e información de los conferencistas. </div>
<div class="form-group text-center">
  <input class="form-control my-4 text-center py-2 m-auto" style="max-width:600px;" type="text" @bind="@message"
    placeholder="Escribe tu mensaje aquí" />
  <button class="btn btn-success mb-4 px-2" style="max-width:200px;" onclick="@GetResponseFromGPT3">Buscar
    respuesta</button>
</div>
<div class="d-flex justify-content-center text-center">
  <div style="min-width:600px; min-height:260px; max-width:700px;"
    class="mt-2container border border-1 rounded-2 p-3 text-center">@generatedText</div>
</div>

@code {
  private string message { get; set; } = "";
  private string generatedText { get; set; } = "La respuesta se mostrará aquí.";
  private readonly HttpClient _httpClient = new HttpClient();

  private async Task GetResponseFromGPT3()
  {
    generatedText = "Buscando respuestas...";

    string azureOpenAIEndpoint = "https://tcd2024-openai.openai.azure.com/";
    string azureOpenAIKey = "dffc8525475949c082e35fc325b9bbe9";
    string searchEndpoint = "https://tcd-search-2024-2.search.windows.net";
    string searchKey = "Hzl8XcrfgWcFcGO5hByqLjCJIibNSzfwX5FHgilokxAzSeCTcfAW";
    string searchIndex = "documentos";
    string deploymentName = "chatbot-assistant-tcd";


    var client = new OpenAIClient(new Uri(azureOpenAIEndpoint), new AzureKeyCredential(azureOpenAIKey));

    var chatCompletionsOptions = new ChatCompletionsOptions()
      {
        Messages =
        {
        new ChatRequestUserMessage(message),
        },
            AzureExtensionsOptions = new AzureChatExtensionsOptions()
            {
              Extensions =
            {
                new AzureCognitiveSearchChatExtensionConfiguration()
            {
                SearchEndpoint = new Uri(searchEndpoint),
                Key = searchKey,
                IndexName = searchIndex,
            },
        }
        },
        DeploymentName = deploymentName
      };

    Response<ChatCompletions> response = client.GetChatCompletions(chatCompletionsOptions);

    ChatResponseMessage responseMessage = response.Value.Choices[0].Message;

    generatedText = $"Mensaje de {responseMessage.Role}:\n";
    generatedText+="===\n";
    generatedText+=responseMessage.Content+"\n";
    generatedText+="===\n";

    generatedText+=$"Información del contexto desde extensiones del chat:\n";
    generatedText+="===\n";
    foreach (ChatResponseMessage contextMessage in responseMessage.AzureExtensionsContext.Messages)
    {
      string contextContent = contextMessage.Content;
      try
      {
        var contextMessageJson = JsonDocument.Parse(contextMessage.Content);
        contextContent = JsonSerializer.Serialize(contextMessageJson, new JsonSerializerOptions()
          {
            WriteIndented = true,
          });
      }
      catch (JsonException)
      { }
      generatedText+=$"{contextMessage.Role}: {contextContent}\n";
    }
    generatedText+="===\n";
  }
}
```
3. Levantar el proyecto

```
dotnet run
```