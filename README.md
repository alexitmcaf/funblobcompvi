# FunBlobCompVI

This project uses Azure Blob Storage and the Azure Computer Vision API to process images uploaded to a Blob Storage container and analyze them using AI. The analysis results are logged and stored for further use.

## Prerequisites

1. **Azure Subscription**
   - You need an active Azure subscription.

2. **Azure CLI**
   - Install Azure CLI to manage Azure resources from your terminal.

3. **Node.js**
   - Install Node.js (version 14 or above).

4. **Azure Functions Core Tools**
   - Install Azure Functions Core Tools to run and debug Azure Functions locally.

## Setup Instructions

### Step 1: Create Azure Resources

1. **Resource Group**
   ```bash
   az group create --name funblobcompvi-rg --location <location>
   ```

2. **Storage Account**
   ```bash
   az storage account create --name funblobcompvi --resource-group funblobcompvi-rg --location <location> --sku Standard_LRS
   ```

3. **Blob Containers**
   - Create containers for input and output files.
   ```bash
   az storage container create --account-name funblobcompvi --name image-input
   az storage container create --account-name funblobcompvi --name image-output
   ```

4. **Computer Vision Service**
   - Create an Azure Computer Vision service.
   ```bash
   az cognitiveservices account create \
     --name funblobcompvi-cv \
     --resource-group funblobcompvi-rg \
     --kind ComputerVision \
     --sku S1 \
     --location <location> \
     --yes
   ```
   - Retrieve the API name and key:
     ```bash
     az cognitiveservices account show --name funblobcompvi-cv --resource-group funblobcompvi-rg --query "endpoint"
     az cognitiveservices account keys list --name funblobcompvi-cv --resource-group funblobcompvi-rg
     ```

### Step 2: Configure Environment Variables

- Set the following environment variables in your local environment or deployment:
  - `COMPUTER_VISION_API_NAME`: Name of the Computer Vision API.
  - `COMPUTER_VISION_API_KEY`: API key for the Computer Vision API.
  - `AZURE_STORAGE_CONNECTION_STRING`: Connection string for the Storage Account.

### Step 3: Install Dependencies

- Run the following command to install the necessary npm packages:
  ```bash
  npm install @azure/functions axios
  ```

### Step 4: Run the Function Locally

1. Start the Azure Functions runtime:
   ```bash
   func start
   ```
2. Upload an image to the `image-input` Blob Storage container. The function will trigger automatically, process the image, and log the results.

### Step 5: Deploy the Function to Azure

1. Deploy your function to Azure:
   ```bash
   func azure functionapp publish <your-function-app-name>
   ```

### Step 6: Monitor Logs

- Use Azure Monitor or `func logs` to view function execution logs and monitor errors.

## `index.js` Function Details

This function triggers whenever a new blob is added to the `image-input` container.

1. **Blob Processing**:
   - Reads the uploaded blob and logs its metadata.

2. **Call Azure Computer Vision API**:
   - Sends the blob content to the Computer Vision API for analysis.
   - Extracts and logs image tags.

3. **Store Analysis Results**:
   - Outputs the analysis results to the `image-output` container.

### Example Code

```javascript
const { app } = require('@azure/functions');
const axios = require('axios');

app.storageBlob('functionstorageBlobTrigger1', {
    path: 'image-input/{name}',
    connection: 'analizeimageblobtriggerc_STORAGE',
    handler: async (blob, context) => {
        try {
            context.log(`Processing blob "${context.triggerMetadata.name}" with size ${blob.length} bytes`);

            const apiUrl = 'https://<<COMPUTER_VISION_API_NAME>>.cognitiveservices.azure.com/vision/v2.0/analyze?visualFeatures=Tags&language=en';
            const apiKey = '<<COMPUTER_VISION_API_KEY>>';

            const response = await axios.post(apiUrl, blob, {
                headers: {
                    'Ocp-Apim-Subscription-Key': apiKey,
                    'Content-Type': 'application/octet-stream',
                },
            });

            context.log("Received response from Computer Vision API:", response.data);

            const tags = response.data.tags.map(tag => tag.name).join(', ');
            context.log(`Image Tags: ${tags}`);

            //context.bindings.outputDocument = JSON.stringify(response.data);

        } catch (error) {
            context.log(`ERROR: ${error.message}`);
            if (error.response) {
                context.log(`ERROR DETAILS: ${JSON.stringify(error.response.data)}`);
            }
        }
    }
});
```

## Troubleshooting

- Ensure the correct API key and name are used.
- Check the blob size and format; ensure it's compatible with the Computer Vision API.
- Use the Azure portal or CLI to diagnose any issues with Blob Storage or the Cognitive Services resource.

## License

This project is licensed under the MIT License. See the LICENSE file for details.
