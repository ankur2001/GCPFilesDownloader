#node js

const { Storage } = require('@google-cloud/storage');
const fs = require('fs');
const path = require('path');

// Replace with the path to your downloaded service account key file
const SERVICE_ACCOUNT_KEY_PATH = './acquired-racer-458623-j7-fe1d75120c56.json';

// Replace with your bucket and object details
const BUCKET_NAME = 'your-bucket-name';
const FILE_NAME = 'your-object-name.ext'; // e.g., report.pdf
const DESTINATION = 'downloaded-file.ext'; // Local file destination

async function downloadFile() {
  // Step 1: Create a Storage client with service account credentials
  const storage = new Storage({
    keyFilename: SERVICE_ACCOUNT_KEY_PATH
  });

  // Step 2: Download the file from GCS
  const options = {
    destination: path.join(__dirname, DESTINATION),
  };

  try {
    await storage.bucket(BUCKET_NAME).file(FILE_NAME).download(options);
    console.log(`File downloaded to ${DESTINATION}`);
  } catch (err) {
    console.error('Error downloading file:', err.message);
  }
}

downloadFile();

====================

# GCPFilesDownloader

<dependencies>
  <dependency>
    <groupId>com.google.auth</groupId>
    <artifactId>google-auth-library-oauth2-http</artifactId>
    <version>1.19.0</version>
  </dependency>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-storage</artifactId>
    <version>2.30.0</version>
  </dependency>
</dependencies>


import com.google.auth.oauth2.AccessToken;
import com.google.auth.oauth2.GoogleCredentials;
import com.google.cloud.storage.Blob;
import com.google.cloud.storage.Storage;
import com.google.cloud.storage.StorageOptions;

import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Collections;

public class GCSFileDownloader {

    private static final String SERVICE_ACCOUNT_KEY_PATH = "path/to/your/service-account.json";
    private static final String BUCKET_NAME = "your-bucket-name";
    private static final String OBJECT_NAME = "your-object-name";
    private static final String DESTINATION_FILE = "downloaded-file.txt";

    public static void main(String[] args) throws IOException {
        // Step 1: Load credentials and get access token
        GoogleCredentials credentials = GoogleCredentials.fromStream(new FileInputStream(SERVICE_ACCOUNT_KEY_PATH))
                .createScoped(Collections.singleton("https://www.googleapis.com/auth/cloud-platform"));
        credentials.refreshIfExpired();
        AccessToken token = credentials.getAccessToken();
        System.out.println("Access Token: " + token.getTokenValue());

        // Step 2: Use the token to access GCS and download a file
        Storage storage = StorageOptions.newBuilder()
                .setCredentials(credentials)
                .build()
                .getService();

        Blob blob = storage.get(BUCKET_NAME, OBJECT_NAME);
        if (blob == null) {
            System.err.println("Object not found in bucket.");
            return;
        }

        try (FileOutputStream outputStream = new FileOutputStream(DESTINATION_FILE)) {
            blob.downloadTo(outputStream);
            System.out.println("File downloaded to: " + DESTINATION_FILE);
        }
    }
}
