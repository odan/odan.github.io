---
title: Reducing Background Audio Noise Using Neural Networks
layout: post
comments: true
published: true
description:
keywords: 
image: 
---

## Introduction

If you've ever tried to record a podcast, an interview, or simply capture some ambient sounds, you'll know the struggle of dealing with unwanted background noise. Fortunately, neural network models have come a long way and can now help us filter out that unwanted noise from our audio clips. In this article, we will look at how to reduce background noise from audio files using a pre-trained neural network model called `arnndn` with `ffmpeg`.

## Requirements

* [FFmpeg](https://www.ffmpeg.org/)
* [Git](https://git-scm.com/downloads)
* [PHP](https://www.php.net/)

## Getting Started

To begin with, we first need to download the necessary neural network model. 
Execute the following command to clone the repository containing the pre-trained models:

```bash
git clone https://github.com/GregorR/rnnoise-models.git
```

After downloading the repository, you're all set to filter the noise from your audio files.

## The ffmpeg Command

```bash
ffmpeg -i in.mp3 -af "arnndn=m='rnnoise-models/somnolent-hogwash-2018-09-01/sh.rnnn'" out.mp3
```

Breaking it down: 

1. **ffmpeg**  - The audio and video processing tool. 
2. **-i in.mp3**  - Specifies the input audio file. Replace `in.mp3` with your filename. 
3. **-af**  - Represents audio filters. 
4. **"arnndn=m='rnnoise-models/somnolent-hogwash-2018-09-01/sh.rnnn'"**  - The filter in use. `arnndn` stands for a neural network denoiser and the model (`m`) is the path to the neural network model we downloaded. 
5. **out.mp3**  - The output file name after noise reduction.

## Automating the Task with PHP

If you have a bunch of `.mp3` audio files that you wish to process, the task can be automated using a simple PHP script:

Filename: `convert.php`

```php

<?php

foreach (glob('*.mp3') as $filename) {
    echo "File: $filename\n";

    exec(
        sprintf(
            'ffmpeg -i "%s" -af "arnndn=m=\'rnnoise-models/somnolent-hogwash-2018-09-01/sh.rnnn\'" "out_%s.mp3"',
            $filename,
            pathinfo($filename, PATHINFO_FILENAME)
        )
    );
}
```

This script will search for all files with the `.mp3` extension in the current directory. 

The `exec` command runs the `ffmpeg` command for each audio file to reduce the background noise and then saves the output in an `.mp3` format, prefixed with `out_`.

To run the script, save it to a `.php` file, place it in the directory with your `.mp3` files, and run it using the PHP CLI.

Note, this ffmpeg command also works with `.m4p` files.

```bash
php convert.php
```

## Conclusion

With advancements in neural network models and tools like `ffmpeg`, it's easier than ever to enhance the quality of our audio recordings by filtering out unwanted noise. Whether you're a podcaster, content creator, or simply someone looking to improve their audio files, this method provides an efficient way to achieve clearer audio.


## Read more

Credits to:

* [How to reduce background audio noise using arnndn (neural network models)](https://onelinerhub.com/ffmpeg/how-to-reduce-background-audio-noise-using-arnndn)