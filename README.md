const { Storage } = require('@google-cloud/storage');
const fs = require('fs');
const path = require('path');

// === CONFIGURATION ===
const GS_URI = 'gs://testgcpdownload1234';
const DEST_FOLDER = './downloads';
const KEY_FILE = './acquired-racer-458623-j7-fe1d75120c56.json';
const START_NUMBER = 3034; // Start from IMG_3034.JPG

// === Extract bucket name from gs:// URI ===
function parseBucketName(gsUri) {
  if (!gsUri.startsWith('gs://')) {
    throw new Error('Invalid GCS URI. It should start with "gs://".');
  }
  const parts = gsUri.replace('gs://', '').split('/');
  return parts[0];
}

async function downloadFromGsUri(gsUri, destinationFolder) {
  const bucketName = parseBucketName(gsUri);
  const storage = new Storage({ keyFilename: KEY_FILE });

  const bucket = storage.bucket(bucketName);
  const [files] = await bucket.getFiles();

  // Sort by filename
  files.sort((a, b) => a.name.localeCompare(b.name));

  if (!fs.existsSync(destinationFolder)) {
    fs.mkdirSync(destinationFolder, { recursive: true });
  }

  for (const file of files) {
    const baseName = path.basename(file.name); // e.g., IMG_3034.JPG
    const match = baseName.match(/^IMG_(\d+)\.JPG$/i);

    if (!match) {
      continue; // Skip if filename doesn't match expected pattern
    }

    const fileNum = parseInt(match[1], 10);
    if (fileNum < START_NUMBER) {
      continue; // Skip files before the start number
    }

    const destPath = path.join(destinationFolder, file.name);
    const dir = path.dirname(destPath);
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }

    console.log(`⬇️  Downloading: ${file.name}`);
    await file.download({ destination: destPath });
  }

  console.log(`✅ Files from IMG_${START_NUMBER}.JPG onward downloaded successfully.`);
}

// === RUN ===
downloadFromGsUri(GS_URI, DEST_FOLDER).catch(console.error);


================================




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
