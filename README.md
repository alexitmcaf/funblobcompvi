# funblobcompvi

create rg, storage account, in blob create containers for input files - image-input(for path), for output files - 
cr computer vision ai service,  get from here COMPUTER_VISION_API_NAME, COMPUTER_VISION_API_KEY
run check logs

index.js

const { app } = require('@azure/functions');
const axios = require('axios');

app.storageBlob('functionstorageBlobTrigger1', {
    path: 'image-input/{name}',
    connection: 'analizeimageblobtriggerc_STORAGE',
    handler: async (blob, context) => {
        try {
            context.log(`Processing blob "${context.triggerMetadata.name}" with size ${blob.length} bytes`);

            // Azure Computer Vision API Details
            const apiUrl = 'https://<<COMPUTER_VISION_API_NAME>>.cognitiveservices.azure.com/vision/v2.0/analyze?visualFeatures=Tags&language=en';
            const apiKey = '<<COMPUTER_VISION_API_KEY>>';

            // Call the Computer Vision API
            const response = await axios.post(apiUrl, blob, {
                headers: {
                    'Ocp-Apim-Subscription-Key': apiKey,
                    'Content-Type': 'application/octet-stream',
                },
            });

            context.log("Received response from Computer Vision API:", response.data);

            // Example: Log tags from the API response
            const tags = response.data.tags.map(tag => tag.name).join(', ');
            context.log(`Image Tags: ${tags}`);

        } catch (error) {
            // Use context.log to report errors
            context.log(`ERROR: ${error.message}`);
            if (error.response) {
                context.log(`ERROR DETAILS: ${JSON.stringify(error.response.data)}`);
            }
        }
    }
});

for output -  context.bindings.outputDocument = JSON.stringify(response.data);
