---
layout: post
title: "Access our Speechlab engine with live stream mode"
author: "AISpeechlab"
categories: resources
tags: [documentation,demo]
image: arctic-4.jpg
---

# How it works

Offline speech recognition server is built based on the Kaldi toolkit and implemented in Python, other scripting languages.

## File type and language supported


| Spoken languages      | Description                                           |
| --------------------- | ----------------------------------------------------- |
| Singapore             | Where user speaks Singapore English                   |
| Code-switch           | Where user speaks English and Mandarin interchangably |
| Mandarin              | Where user speaks Mandarin                            |

User can upload file with size up to 500MB each time
For all language models, we support the following file types: .mp3, .mp4, .wav

## Recording guidelines

For best results, make sure that your audio documents also meet these requirements:  

    * Channel audio: 16 kHz (recommended) sample rate  
    * Recording Guidelines: When transcribing audio, make sure that the audio recordings you provide to the transcription service meet these recording guidelines


**Record in a quiet environment.**  
Background noise degrades the performance of the Speech Lab engines.

**Speak clearly.**  
Make sure that you speak such that your voice can be clearly heard in the recording. Avoid speaking too softly.

**Speak at a consistent volume.**  
Make sure that that the speech segments in your audio document have a consistent volume. A recording with widely varying volume levels degrades the performance of the Speech Lab engines.

**Mono channel audio.**  
The Live Transcriber currently support only single-track audio files. If your audio file uses more than one audio channel, the transcription service will only use the first audio channel. For better performance, downmix your audio document to mono audio before attempting to run it through our transcription engines.

**Record at 16kHz.**  
The Live Transcriber is optimized to transcribed audio sampled at 16kHz. Transcribing audio with a sampling rate that is significantly higher or lower than 16kHz may degrade the performance of the Speech Lab engines.

**Take turns speaking.**  
If your recording includes multiple speakers, make sure that they don’t speak over each other. Overlapping speech degrades the performance of the Speech Lab engines.


**Use models that support the spoken language.**  
Use the model that supports the spoken language in the audio document. Speech in a language that is not supported by the model selected when beginning transcription will be incorrectly transcribed.


# Websocket-based client-server protocol  

## Opening a session  

To open a session, connect to the specified server websocket address (e.g. ws://SERVER:PORT/client/ws/speech?content-type=?token=). The server assumes by default that incoming audio is sent using 16 kHz, mono, 16-bit little-endian format. This can be overridden using the 'content-type' request parameter. The content type has to be specified using GStreamer 1.0 caps format, e.g. to send 44100 Hz mono 16-bit data, use: "audio/x-raw, layout=(string)interleaved, rate=(int)44100, format=(string)S16LE, channels=(int)1". This needs to be url-encoded of course, so the actual request is something like:

```
ws://SERVER:PORT/client/ws/speech?content-type=audio/x-raw,+layout=(string)interleaved,+rate=(int)44100,+format=(string)S16LE,+channels=(int)1?token=<YOUR_ACCESS_TOKEN>
```

Audio can also be encoded using any codec recognized by GStreamer (assuming the needed packages are installed on the server). GStreamer should recognize the container and codec automatically from the stream, you don't have to specify the content type.

## Sending audio
Speech should be sent to the server in raw blocks of data, using the encoding specified when session was opened. It is recommended that a new block is sent at least 4 times per second (less frequent blocks would increase the recognition lag). Blocks do not have to be of equal size. Server closes the connection itself when all recognition results have been sent to the client. In order to process a new audio stream, a new websocket connection has to be created by the client.

## Reading results
Server sends recognition results and other information to the client using the JSON format. The response can contain the following fields:  

status -- response status (integer), see codes below
message -- (optional) status message
result -- (optional) recognition result, containing the following fields:
hypotheses - recognized words, a list with each item containing the following:
transcript -- recognized words
confidence -- (optional) confidence of the hypothesis (float, 0..1)
final -- true when the hypothesis is final, i.e., doesn't change any more


The following status codes are currently in use:  

0 -- Success. Usually used when recognition results are sent  
2 -- Aborted. Recognition was aborted for some reason.  
1 -- No speech. Sent when the incoming audio contains a large portion of silence or non-speech.  
9 -- Not available. Used when all recognizer processes are currently in use and recognition cannot be performed.  


Websocket is always closed by the server after sending a non-zero status update.

Examples of server responses:  
  ```
  {"status": 9}{"status": 0, "result": {"hypotheses": [{"transcript": "see on"}], "final": false}}  
  {"status": 0, "result": {"hypotheses": [{"transcript": "see on teine lause."}], "final": true}}
  ```

Server segments incoming audio on the fly. For each segment, many non-final hypotheses, followed by one final hypothesis are sent. Non-final hypotheses are used to present partial recognition hypotheses to the client. A sequence of non-final hypotheses is always followed by a final hypothesis for that segment. After sending a final hypothesis for a segment, server starts decoding the next segment, or closes the connection, if all audio sent by the client has been processed. Client is responsible for presenting the results to the user in a way suitable for the application.


# Alternative usage through a HTTP API
One can also use the server through a very simple HTTP-based API. This allows to simply send audio via a PUT or POST request to http://server:port/client/dynamic/recognize and read the JSON output. The HTTP API simply takes a PUT request with header Content-Type=audio/x-wav and an audio file in either wav or mp3 as body and nothing else


Example of sending audio to server  
```
$ curl  -T test/data/english_test.wav  "http://localhost:8888/client/dynamic/recognize"
```

Output
```
{"status": 0, "hypotheses": [{"utterance": "one two or three you fall five six seven eight. [noise]."}], "id": "7851281f-e187-4c24-9b58-4f3a5cba3dce" }
```

The HTTP API supports chunked transfer encoding which means that server can read and decode an audio stream before it is complete.

Send audio using chunked transfer encoding at an audio byte rate; you can see from the worker logs that decoding starts already when the first chunks have been received:
```
curl -v -T test/data/english_test.raw -H "Content-Type: audio/x-raw-int; rate=16000" --header "Transfer-Encoding: chunked" --limit-rate 32000  "http://localhost:8888/client/dynamic/recognize"
```


Output
```
{"status": 0, "hypotheses": [{"utterance": "one two or three you fall five six seven eight. yeah."}], "id": "4e4594ee-bdb2-401f-8114-41a541d89eb8"}

```
