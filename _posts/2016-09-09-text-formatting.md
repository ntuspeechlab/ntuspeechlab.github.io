---
layout: post
title: "Access our Speechlab engine with batch mode"
author: "Ly, Duc"
categories: sample
tags: [sample]
image: arctic-1.jpg
---

# How it works

Offline speech recognition server is built based on the Kaldi toolkit and implemented in Python, other scripting languages.

## Process of transcribing audio

The above picture illustrates how the offline decoding system works. The audio input will be processed through steps

**Step 1: Resample the audio file**  
The audio needs to be split into mono channels, and sample rate that match with the trained model.
Tools used: Soxi/ffmpeg

**Step 2: Detect the speech in the input**  
Speaker diarisation (or diarization) is the process of partitioning an input audio stream into homogeneous segments according to the speaker identity.
Output of this process is the segment file (.seg), including the speaker id, segment that including speech, and start/end time
Tools used: LIUM 8.4.1

**Step 3: Convert the audio to proper format (kaldi format)**  
To process further by the kaldi toolkit, the audio data and segment file will be parse to kaldi script, to convert to kaldi proper format.
Tools used: Kaldi scripts

**Step 4: Extract features from the input**  
Extract the features from the kaldi format, mfcc and iVector features.
Tools used: Kaldi scripts

**Step 5: Decode/Generate the transcription**  
Features extracted from previous step will be parsed to kaldi toolkit, with our trained model, to generate the transcription in ctm/stm format.
Furthermore, transcription are also converted to different formats, support different requests from user: like TextGrid, csv, text.

The file output will be sent to public folder, user could have other post-processing like converting to their required format, sending to other modules (language understanding, adding sentence unit, etc.)

The system will process input files sequentially. The ‘file_name’ of input audio files will be normalized into ‘file_id’. The output folder will have the following structure:

```
/path/to/the/output/folder/
.
├── <file-id-1>  
│   ├── <file-id-1>.<model_name>.ctm  
│   ├── <file-id-1>.<model_name>.srt  
│   ├── <file-id-1>.<model_name>.stm  
│   ├── <file-id-1>.<model_name>.TextGrid  
│   └── <file-id-1>.<model_name>.txt  
├── <file-id-2>  
│   ├── <file-id-2>.<model_name>.ctm  
│   ├── <file-id-2>.<model_name>.srt  
│   ├── <file-id-2>.<model_name>.stm  
│   ├── <file-id-2>.<model_name>.TextGrid  
│   └── <file-id-2>.<model_name>.txt  
└── <file-id-3: eg: 8khz-testfile>  
    ├── 8khz-testfile.<model_name>.ctm  
    ├── 8khz-testfile.<model_name>.srt  
    ├── 8khz-testfile.<model_name>.stm  
    ├── 8khz-testfile.<model_name>.TextGrid  
    └── 8khz-testfile.<model_name>.txt  
```
_* Other file types can be existed._

Information extraction from the output:  
*.ctm file including word and start time, end time of each word  
*.srt, *.stm, *.textgrid including segments (sentences) with start time, end time.  
*.txt including the whole transcription.  


## File type and language supported


| Spoken languages      | Description                                           |
| --------------------- | ----------------------------------------------------- |
| Singapore             | Where user speaks Singapore English                   |
| Code-switch           | Where user speaks English and Mandarin interchangably |
| Mandarin              | Where user speaks Mandarin                            |

User can upload file with size up to 500MB each time
For all language models, we support the following file types: .mp3, .mp4, .wav


## Recording notes

No matter what the input audio format, our offline system can process to down/up sample the audio file to the 16khz sampling rate, 16 bit rate, and mono channel to process. Our system performs best with audio in 16Khz sampling rate, 16 bit rate and close talk or telephony with clear and clean speech.

## Links

[link to gitbook!](https://maitrungduc.gitbook.io/ai-speechlab/guide/offline/how-it-works).

# Usage

We provide user 2 way of using our offline system:
* Using HTTP request  
which provides modern and standard interface. User can call our HTTP endpoints using curl or other tools like Postman. Our API can also be integrated as third party service.
* The second way to use our offline system is through our web based application.  
We provide user an intuitive and easy UI to interact and explore how our offline engine works. Check out our next sections to see how to use our system.

## Using HTTP Request

### POST | Register
```
https://gateway.speechlab.sg/auth/register
```

This endpoint allows you to register an account


* Request  

    | Body Parameters       | Type                  | Description            |
    | --------------------- | --------------------- | ---------------------- |
    | email - REQUIRED      | string                | Account's email        |
    | password - REQUIRED   | string                | Account's password     |

* Response  
    * 200: OK  
    Register successfully. You should check your inbox for verification email

    ```json
    {
        "role": "user",
        "isVerified": false,
        "_id": "user_id",
        "email": "user_email",
        "createdAt": "created date",
        "message": "Please check your inbox for verification email"
    }
    ```

    * 400: Bad Request  
    Your email or password is not valid  

    ```json-doc
    {
        "statusCode": 400,
        "message": [
            "password must be longer than or equal to 6 characters",
            "password should not be empty",
            "password must be a string"
        ],
        "error": "Bad Request"
    }
    ```

After successfully register your account, there will be an email sent to your inbox to verify your email address. You need to click into verification link which is sent along with the email.  

### POST | Login
```
https://gateway.speechlab.sg/auth/login
```

This endpoint allows you to login and get an account token

* Request  

    | Body Parameters       | Type                  | Description            |
    | --------------------- | --------------------- | ---------------------- |
    | email - REQUIRED      | string                | Account's email        |
    | password - REQUIRED   | string                | Account's password     |

* Response  
    * 200: OK  
    Login successfully  

    ```json
    {
        "accessToken": "Your token"
    }
    ```

    * 404: Not Found  
    Email or password is incorrect  

    ```json-doc
    {
        "statusCode": 401,
        "message": "Unauthorized"
    }
    ```

### POST | Submit a job

```
https://gateway.speechlab.sg/speech
```

This endpoint allows you to login and get an account token

* Request  

    * Headers  

    | Header Parameters        | Type      | Description                                             |
    | ---------------------    | --------- | ------------------------------------------------------- |
    | Authorization - REQUIRED | string    | Access token. Format: Bearer <your token after login>   |

    * Form Data Paramters  

    | Form Data Parameters  | Type    | Description                                                                                                                                 |
    | --------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
    | file - REQUIRED       | object  | File object (binary)                                                                                                                        |
    | lang - REQUIRED       | string  | must be one of: ['english', 'mandarin', 'malay', 'english-mandarin', 'english-malay']                                                       |
    | queue - optional      | string  | Queue where your job will be running in                                                                                                     |
    | audioType - optional  | string  | One of: ['closetalk', 'telephony']                                                                                                          |
    | audioTrack - optional | string  | One of: ['single', 'multi']                                                                                                                 |
    | webhook - optional    | string  | API endpoint to receive update when your speech is ready (good way to get update of a speech, see below). Eg: http://your-api.com/status    |

* Response  
    * 200: OK  
    Submited job successfully  

    ```json-doc
    {
        "result": null,
        "status": "created",
        "formats": [
            "stm",
            "srt",
            "TextGrid"
        ],
        "sampling": "16khz",
        "lang": "english",
        "name": "Note",
        "_id": "5f11c797b6d18b0029b60975", // Note this ID
        "queue": "normal",
        "createdAt": "2020-07-17T15:45:27.882Z"
    }
    ```

    * 403: Forbidden  
    This status can be one one:  
        * Your email hasn't been verified. Please check your inbox for verification email  
        * You're not allowed to submit job to this queue  
        * Your account has been blocked. Please contact our administration for further detail.  

    ```json-doc
    {
        "statusCode": 403,
        "message": "You haven't verify your email yet. Please check your inbox.",
        "error": "Forbidden"
    }
    ```

    * 404: Not Found  
    No speech found with provided ID  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Speech not found",
        "error": "Not Found"
    }
    ```

* Note: If you can provide audioType and audioTrack that best describe your audio file, we'll try to use our best-fit model to give you better result.

After request successfully, in response returned, you'll see a field name _id, this is ID of your speech, we also call it SpeechID. Notice this for further requests.  

About **webhook**: an audio may take long time to decode (depends on its duration, type,...), and the processing time is not exact, not same even if you submit 2 identical jobs. So instead of keep calling us to get the status, You can provide your HTTP endpoint in webhook field and we'll send your audio's status, and when status is done you just need to call our endpoint to download your transcription  

About **queue**: a queue is where your job will be running in, each queue will have some workers to pick up your job and process, by default your job will be submitted to normal queue and this queue is shared among others, because of that sometimes your job will have to be queued until our workers available. If you want to have dedicated queue to not share with others, contact our support to get in detail.  


### GET | Get job status

```
https://gateway.speechlab.sg/speech/:id
```

This api is to retrieve speech's status. If you mean to call this api many times to get speech's status, consider passing webhook when submitting your job as described in above section

* Request  

    * Path parameters

    | Parameters               | Type      | Description       |
    | ---------------------    | --------- | ----------------- |
    | id - REQUIRED            | string    | Speech's ID       |

    * Headers

    | Header Parameters        | Type      | Description                                             |
    | ---------------------    | --------- | ------------------------------------------------------- |
    | Authorization - REQUIRED | string    | Access token. Format: Bearer <your token after login>   |


* Response  
    * 200: OK  
    Login successfully  

    ```json
    {
        "status": "processing",
        "formats": [
            "stm",
            "srt",
            "TextGrid"
        ],
        "sampling": "16khz",
        "lang": "english",
        "name": "Note",
        "_id": "5f11c797b6d18b0029b60975",
        "queue": "normal",
        "input": [
            {
                "isSubmitted": true,
                "errorCode": null,
                "status": "processing",
                "progress": [{
                    "content": "Decoding",
                    "createdAt": "2020-07-17T15:45:27.941Z"
                }],
                "_id": "5f11c798b6d18b0029b60977",
                "file": {
                    "_id": "5f11c797b6d18b0029b60976",
                    "originalName": "james-test.wav",
                    "mimeType": "audio/wave",
                    "filename": "5f11c797b6d18b0029b60976.wav",
                    "size": 187404,
                    "duration": 5.855,
                    "createdAt": "2020-07-17T15:45:27.941Z"
                }
            }
        ],
        "createdAt": "2020-07-17T15:45:27.882Z",
        "sourceFile": {
            "_id": "5f11c797b6d18b0029b60976",
            "originalName": "james-test.wav",
            "mimeType": "audio/wave",
            "filename": "5f11c797b6d18b0029b60976.wav",
            "size": 187404,
            "duration": 5.855,
            "createdAt": "2020-07-17T15:45:27.941Z"
        }
    }
    ```

    * 403: Forbidden  
    This status can be one of:  
    - Your haven't verified your account  
    - Your account has been blocked  
    ```
    {
        "statusCode": 403,
        "message": "You haven't verify your email yet. Please check your inbox.",
        "error": "Forbidden"
    }
    ```

    * 404: Not Found  
    No speech found with provided ID  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Speech not found",
        "error": "Not Found"
    }
    ```

### GET | Download transcription
```
https://gateway.speechlab.sg/speech/:id/result
```

This API returns transcription when speech's status is **done**

* Request  

    * Path parameters

    | Parameters               | Type      | Description       |
    | ---------------------    | --------- | ----------------- |
    | id - REQUIRED            | string    | Speech's ID       |

    * Headers

    | Header Parameters        | Type      | Description                                             |
    | ---------------------    | --------- | ------------------------------------------------------- |
    | Authorization - REQUIRED | string    | Access token. Format: Bearer <your token after login>   |


* Response  
    * 200: OK  

    ```json
    {
        url: "a downloadable link"
    }
    ```

    * 403: Forbidden  
    This status can be one of:  
    - Your haven't verified your account  
    - Your account has been blocked  
    ```
    {
        "statusCode": 403,
        "message": "You haven't verify your email yet. Please check your inbox.",
        "error": "Forbidden"
    }
    ```

    * 404: Not Found  
    No speech found with provided ID  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Speech not found",
        "error": "Not Found"
    }
    ```

### POST | Change account's password
```
https://gateway.speechlab.sg/auth/change-password
```

This endpoint allows you to login and get an account token

* Request  

    | Body Parameters               | Type                  | Description                |
    | ----------------------------- | --------------------- | -------------------------- |
    | email - REQUIRED              | string                | Account's email            |
    | currentPassword - REQUIRED    | string                | Account's current password |
    | newPassword - REQUIRED        | string                | Account's new password     |
    | confirmNewPassword - REQUIRED | string                | Account's new password     |


* Response  
    * 200: OK  
    Login successfully  

    ```json
    {
        "message": "Password changed successfully"
    }
    ```

    * 400: Bad Request  
    Your body parameters are incorrect  

    ```json-doc
    {
        "statusCode": 400,
        "message": [
            "email must be longer than or equal to 6 characters",
            "email must be an email"
        ],
        "error": "Bad Request"
    }
    ```

    * 404: Not Found  
    No account found with provided email  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Account not found",
        "error": "Not Found"
    }
    ```

### POST | Forgot account's password
```
https://gateway.speechlab.sg/auth/forgot-password
```

This API is used to reset account password

* Request  

    | Body Parameters       | Type                  | Description            |
    | --------------------- | --------------------- | ---------------------- |
    | email - REQUIRED      | string                | Account's email        |


* Response  
    * 200: OK  
    Request to reset password sent successfully, check your inbox to get reset code  

    ```json
    {
        "message": "Please check your inbox for reset password code"
    }
    ```

    * 400: Bad Request  
    Your body parameters are incorrect  

    ```json-doc
    {
        "statusCode": 400,
        "message": [
            "email must be longer than or equal to 6 characters",
            "email must be an email"
        ],
        "error": "Bad Request"
    }
    ```

    * 404: Not Found  
    No account found with provided email  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Account not found",
        "error": "Not Found"
    }
    ```

After sending this API, there's an email will be sent to your email address which contains a code used to reset password (see the API below)

### POST | Reset password
```
https://gateway.speechlab.sg/auth/reset-password
```

* Request  

    | Body Parameters               | Type                  | Description                |
    | ----------------------------- | --------------------- | -------------------------- |
    | email - REQUIRED              | string                | Account's email            |
    | currentPassword - REQUIRED    | string                | Account's current password |
    | newPassword - REQUIRED        | string                | Account's new password     |
    | confirmNewPassword - REQUIRED | string                | Account's new password     |

* Response  
    * 200: OK  
    Successfully reset your account's password  

    ```json
    {
        "message": "Your password has been reset"
    }
    ```

    * 400: Bad Request  
    Your body parameters are incorrect  

    ```json-doc
    {
        "statusCode": 400,
        "message": [
            "email must be longer than or equal to 6 characters",
            "email must be an email"
        ],
        "error": "Bad Request"
    }
    ```

    * 404: Not Found  
    No account found with provided email  

    ```json-doc
    {
        "statusCode": 404,
        "message": "Email not found or code is incorrect",
        "error": "Not Found"
    }
    ```

    * 406: Not Acceptable  
    Your newPassword and confirmNewPassword are not matched  

    ```json-doc
    {
        "statusCode": 406,
        "message": "New passwords are not matched each other",
        "error": "Not Acceptable"
    }
    ```


### POST | Send verification email
```
https://gateway.speechlab.sg/auth/verify
```

Verification email is sent right after account registration, and will be valid for 12 hours. This api endpoint is used in case you want to send new verification email

* Request  

    | Header Parameters        | Type      | Description                                             |
    | ---------------------    | --------- | ------------------------------------------------------- |
    | Authorization - REQUIRED | string    | Access token. Format: Bearer <your token after login>   |

* Response  
    * 200: OK  
    Login successfully  

    ```json
    {
        "accessToken": "Please check your inbox for verification email"
    }
    ```

    * 422: Unprocessable Entity  
    Your account has been verified before  

    ```json-doc
    {
        "statusCode": 422,
        "message": "Account has been verified before"
    }
    ```

### POST | Create an application
```
https://gateway.speechlab.sg/applications
```

This endpoint is used to create a third-party application that make uses of Gateway APIs.
**Note**: if success, app **secret** will only be shown once, be sure to save it somewhere privately

* Request  

    * Headers

    | Header Parameters        | Type      | Description                                             |
    | ---------------------    | --------- | ------------------------------------------------------- |
    | Authorization - REQUIRED | string    | Access token. Format: Bearer <your token after login>   |

    * Body

    | Body Parameters       | Type                  | Description             |
    | --------------------- | --------------------- | ----------------------- |
    | name - REQUIRED       | string                | Your application's name |

* Response  
    * 200: OK  
    Login successfully  

    ```json-doc
    {
        "_id": "5f8d581e87bf680b226d3cc1", // App ID
        "name": "Test App",
        "key": "5f8d581e87bf680b226d3cc0", // Key ID (used to get public key to verify JWT token)
        "createdAt": "2020-10-19T09:10:54.482Z",
        "secret": "41d29bb4c52389441c1647764de94493c16a8c42e9529bdaa7502cccc7f111e7" // App secret
    }
    ```

### POST | Get JWT Public Key
```
https://gateway.speechlab.sg/**keys/:keyId**
```

This endpoints return JWT public key, third party will use this key in order to verify JWT token generated from API gateway

* Request  

    | Body Parameters       | Type       | Description                                                  |
    | --------------------- | ---------- | ------------------------------------------------------------ |
    | KeyId - optional      | string     | Key ID returned after creating application (see above)       |

* Response  
    * 200: OK  
    Login successfully  

    ```json
        public key in file
    ```



{% highlight js %}
See more about websocket protocol
{% endhighlight %}

Another option is to embed your code through [Gist](https://en.support.wordpress.com/gist/).

## Useful Resources (formatting)

More information on Markdown can be found at the following links:

- [Markdown Here Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet#code)
- [Quick Markdown Example](http://www.unexpected-vortices.com/sw/rippledoc/quick-markdown-example.html)
- [Markdown Basics](https://daringfireball.net/projects/markdown/basics)
- [GitHub Flavoured Markdown Spec](https://github.github.com/gfm/)
- [Basic writing and formatting syntax](https://help.github.com/articles/basic-writing-and-formatting-syntax/#lists)
