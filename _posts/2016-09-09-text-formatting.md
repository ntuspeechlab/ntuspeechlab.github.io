---
layout: post
title: "Access our Speechlab engine with batch mode"
author: "Duc"
categories: sample
tags: [sample]
image: arctic-1.jpg
---

# How it works

Offline speech recognition server is built based on the Kaldi toolkit and implemented in Python, other scripting languages.

## Process of transcribing audio

The above picture illustrates how the offline decoding system works. The audio input will be processed through steps

### Step 1: Resample the audio file
The audio needs to be split into mono channels, and sample rate that match with the trained model.
Tools used: Soxi/ffmpeg

### Step 2: Detect the speech in the input
Speaker diarisation (or diarization) is the process of partitioning an input audio stream into homogeneous segments according to the speaker identity.
Output of this process is the segment file (.seg), including the speaker id, segment that including speech, and start/end time
Tools used: LIUM 8.4.1

### Step 3: Convert the audio to proper format (kaldi format)
To process further by the kaldi toolkit, the audio data and segment file will be parse to kaldi script, to convert to kaldi proper format.
Tools used: Kaldi scripts

### Step 4: Extract features from the input
Extract the features from the kaldi format, mfcc and iVector features.
Tools used: Kaldi scripts

### Step 5: Decode/Generate the transcription
Features extracted from previous step will be parsed to kaldi toolkit, with our trained model, to generate the transcription in ctm/stm format.
Furthermore, transcription are also converted to different formats, support different requests from user: like TextGrid, csv, text.

The file output will be sent to public folder, user could have other post-processing like converting to their required format, sending to other modules (language understanding, adding sentence unit, etc.)

The system will process input files sequentially. The ‘file_name’ of input audio files will be normalized into ‘file_id’. The output folder will have the following structure:

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

*Other file types can be existed.
Information extraction from the output:
*.ctm file including word and start time, end time of each word
*.srt, *.stm, *.textgrid including segments (sentences) with start time, end time.
*.txt including the whole transcription.


## File type and language supported

Languages
Description
Singapore Code Switch

Mandarin

Singapore English
Spoken languages      | Description
--------------------- | --------------------:
Singapore             | Where user speaks Singapore English
Code-switch           | Where user speaks English and Mandarin interchangably
Mandarin              | Where user speaks Mandarin

User can upload file with size up to 500MB each time
For all language models, we support the following file types: .mp3, .mp4, .wav


## Recording notes

No matter what the input audio format, our offline system can process to down/up sample the audio file to the 16khz sampling rate, 16 bit rate, and mono channel to process. Our system performs best with audio in 16Khz sampling rate, 16 bit rate and close talk or telephony with clear and clean speech.

## Links

[link to gitbook!](https://maitrungduc.gitbook.io/ai-speechlab/guide/offline/how-it-works).

# Usage

We provide user 2 way of using our offline system:
* Using HTTP request which provides modern and standard interface. User can call our HTTP endpoints using curl or other tools like Postman. Our API can also be integrated as third party service.
* The second way to use our offline system is through our web based application. We provide user an intuitive and easy UI to interact and explore how our offline engine works. Check out our next sections to see how to use our system.

## Using HTTP Request

POST | Login
```
https://gateway.speechlab.sg/auth/login
```

This endpoint allows you to login and get an account token

* Request
Body Parameters       | Type                  | Description 
--------------------- | :-------------------: | :-------------------- | --------------------:
email - REQUIRED      | string                | Account's email
password - REQUIRED   | string                | Account's password 

* Response
200: OK
Login successfully

```json
{
    "accessToken": "Your token"
}
```

404: Not Found
Email or password is incorrect

```json-doc
{
    "statusCode": 401,
    "message": "Unauthorized"
}
```


## Code and Syntax Highlighting

Code blocks are part of the Markdown spec, but syntax highlighting isn't. However, many renderers - like GitHub or most Jekyll themes - support syntax highlighting. Which languages are supported and how those language names should be written will vary from renderer to renderer. You can find the full list of supported programming languages [here](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers). Also, it is possible to do `inline code blocks`, by wrapping the text in ` ` ` quotations.

```
No language indicated, so no syntax highlighting.
```

```ruby
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Another option is to embed your code through [Gist](https://en.support.wordpress.com/gist/).

## Unordered and Numbered Lists

You can make an unordered and nested list by preceding one or more lines of text with `-`, `*`, or `+`, and indenting sublists. The following lists show the full range of possible list formats.

* List item one
    * List item one
        * List item one
        * List item two
        * List item three
        * List item four
    * List item two
    * List item three
    * List item four
* List item two
* List item three
* List item four

Numbered lists are made by using numbers instead of bullet points.

1. List item one
    1. List item one
        1. List item one
        2. List item two
        3. List item three
        4. List item four
    2. List item two
    3. List item three
    4. List item four
2. List item two
3. List item three
4. List item four

## MathJax Example

The [Schrödinger equation](https://en.wikipedia.org/wiki/Schr%C3%B6dinger_equation) is a partial differential equation that describes how the quantum state of a quantum system changes with time:

$$
i\hbar\frac{\partial}{\partial t} \Psi(\mathbf{r},t) = \left [ \frac{-\hbar^2}{2\mu}\nabla^2 + V(\mathbf{r},t)\right ] \Psi(\mathbf{r},t)
$$

[Joseph-Louis Lagrange](https://en.wikipedia.org/wiki/Joseph-Louis_Lagrange) was an Italian mathematician and astronomer who was responsible for the formulation of Lagrangian mechanics, which is a reformulation of Newtonian mechanics.

$$ \frac{\mathrm{d}}{\mathrm{d}t} \left ( \frac {\partial  L}{\partial \dot{q}_j} \right ) =  \frac {\partial L}{\partial q_j} $$

## Tables

Title 1               | Title 2               | Title 3               | Title 4
--------------------- | :-------------------: | :-------------------- | --------------------:
lorem                 | lorem ipsum           | lorem ipsum dolor     | lorem ipsum dolor sit
lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit
lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit
lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit | lorem ipsum dolor sit

## Embedding

Plenty of social media sites offer the option of embedding certain parts of their site on your own site, such as YouTube and Twitter:

<iframe width="560" height="315" src="https://www.youtube.com/embed/mthtn1X4eUY" frameborder="0" allowfullscreen></iframe>

<a class="twitter-grid" data-partner="tweetdeck" href="https://twitter.com/paululele/timelines/755079130027352064">New Collection</a> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Inline HTML elements

HTML defines a long list of available inline tags, which you can mix with Markdown if you like. A complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

## Useful Resources

More information on Markdown can be found at the following links:

- [Markdown Here Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet#code)
- [Quick Markdown Example](http://www.unexpected-vortices.com/sw/rippledoc/quick-markdown-example.html)
- [Markdown Basics](https://daringfireball.net/projects/markdown/basics)
- [GitHub Flavoured Markdown Spec](https://github.github.com/gfm/)
- [Basic writing and formatting syntax](https://help.github.com/articles/basic-writing-and-formatting-syntax/#lists)
