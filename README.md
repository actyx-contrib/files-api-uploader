# IMPORTANT NOTICE

**This repository has been archived and is no longer maintained. You are welcome to copy the code into your own projects and continue using it at your own discretion and risk.**

# files-api-uploader
With Actyx v2.8.0, we've released the `Files API` that provides for uploading to, retrieving from and naming files in Actyx. Shout-out to @wngr who made that happen :pray:. 

For details, please refer to the reference in the [develveloper documentation](https://developer.actyx.com/docs/reference/files-api)

This repository contains a simple app to upload (multiple) files to Actyx to get you started. Hope that helps.

## 📄 Files API

### ✨ Capabilities
* Files can be uploaded as `multipart/formdata`. 
* To make uploading more convenient than using `curl`, this repository contains a sample uploader HTML page to allow for uploading files and folder hierarchies more easily.
* Uploading a file returns a content identifier (`CID`), which is a unique hash based on the file's content.
* Files can be accessed either via
  `http://localhost:4454/api/v2/files/<cid>/<optional path>` or via
  `http://<cid>.actyx.localhost::4454/<optional path>`. This enables serving
  webapps from the content root conveniently.
* The Actyx Naming Service (`ANS`) allows you to attach a name to a `CID` and access the content using that name through `http://<name>.actyx.localhost:4454/<optional path>`. You can also update that name to point to a different `CID`, allowing you to roll out new versions of a file/app.
* The authentication token for Actyx API access can be provided through the `?token=...` query parameter.

### 🚶🏻‍♂️ Walk-Through
1. Get an authentication token using an Actyx manifest and store it in the `AUTH_TOKEN` variable. Uses `curl` to make the request and `jq` to parse the result.
```bash
$ export AUTH_TOKEN=$(echo '
{
    "appId": "com.example.actyx-v1-pond",
    "displayName": "V1 Pond API compatible with V2 store",
    "version": "1.0"
}
' \
| curl \
    -s -X "POST" \
    -d @- \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    http://localhost:4454/api/v2/auth \
| jq -r .token)
```

2. Upload the attached file uploader HTML.
```bash
$ curl -X POST \
   -H "Authorization: Bearer $AUTH_TOKEN" \
   -F file=@upload_files.html \
   http://localhost:4454/api/v2/files

bafybeiaqela5uulk5qycsxncdtv36stuowojyuso3dnwlaninksco5cfpi # Returns the CID of the uploaded file
```
3. Assign a name to the `CID` returned in the previous step, with `uploader` being the name to assign. You'll see `HTTP/1.1 200 OK` somewhere in the output if the upload went through as expected.
```bash
$ echo bafybeiaqela5uulk5qycsxncdtv36stuowojyuso3dnwlaninksco5cfpi | \
  curl -s -X PUT \
  -d @- \
  -H "Authorization: Bearer $AUTH_TOKEN"  \
  http://localhost:4454/api/v2/files/uploader -v
```
4. Open the uploader using the name you assigned earlier, http://uploader.actyx.localhost:4454/.
![image](https://user-images.githubusercontent.com/189410/140385019-60db97b2-8374-4d35-8aa2-dde98ac877d2.png)

5. Upload files or SPA folders as you see fit, noting their `CID`s in case you want to name them as well. Use drag'n'drop to add files, enter your auth token (see step 1) in the corresponding field and click `upload`.
![image](https://user-images.githubusercontent.com/189410/140385146-8c6ef8ae-1a38-4f80-b3bc-66d42bbad4be.png)

6. You can now go ahead, add new content again and/or assign names to existing `CID`s as described before (or make a PR implementing naming against this repo 😏), e.g.
```bash
echo bafybeieod6ngzponojuau3vwxshptnwqcdoa436obbgv3z2jluxzlc4wqy | \
  curl -s -X PUT \
  -d @- \
  -H "Authorization: Bearer $AUTH_TOKEN"  \
  http://localhost:4454/api/v2/files/hello
```

7. After that, you can access the file through the attached name on any Actyx node in the swarm _locally_
![image](https://user-images.githubusercontent.com/189410/140385228-37bf58c0-7a8d-4c74-a0b0-4fb15b4a40a4.png)


