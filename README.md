# ADetailer - Fork with class name selection.

ADetailer is a extension for stable diffusion webui, similar to Detection Detailer, except it uses ultralytics instead of the mmdet.

## What is different in this fork?

This fork adds support for yolo8 models that include multiple class names, such as the original yolo8 models from ultralytics. A particular advantage of such models is that they can be trained to distinguish very similar objects.

![image](https://github.com/borderlineinteractive/adetailer/blob/main/adetailer.JPG)

## Install

(from Mikubill/sd-webui-controlnet)

1. Open "Extensions" tab.
2. Open "Install from URL" tab in the tab.
3. Enter `https://github.com/Bing-su/adetailer.git` to "URL for extension's git repository".
4. Press "Install" button.
5. Wait 5 seconds, and you will see the message "Installed into stable-diffusion-webui\extensions\adetailer. Use Installed tab to restart".
6. Go to "Installed" tab, click "Check for updates", and then click "Apply and restart UI". (The next time you can also use this method to update extensions.)
7. Completely restart A1111 webui including your terminal. (If you do not know what is a "terminal", you can reboot your computer: turn your computer off and turn it on again.)

You can now install it directly from the Extensions tab.

![image](https://i.imgur.com/g6GdRBT.png)

You **DON'T** need to download any model from huggingface.

## Options

| Model, Prompts                    |                                                                                   |                                                   |
| --------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------- |
| ADetailer model                   | Determine what to detect.                                                         | `None` = disable                                  |
| ADetailer prompt, negative prompt | Prompts and negative prompts to apply                                             | If left blank, it will use the same as the input. |
| Skip img2img                      | Skip img2img. In practice, this works by changing the step count of img2img to 1. | img2img only                                      |

| Detection                            |                                                                                              |              |
| ------------------------------------ | -------------------------------------------------------------------------------------------- | ------------ |
| Detection model confidence threshold | Only objects with a detection model confidence above this threshold are used for inpainting. |              |
| Mask min/max ratio                   | Only use masks whose area is between those ratios for the area of the entire image.          |              |
| Mask only the top k largest          | Only use the k objects with the largest area of the bbox.                                    | 0 to disable |

If you want to exclude objects in the background, try setting the min ratio to around `0.01`.

| Mask Preprocessing              |                                                                                                                                     |                                                                                         |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Mask x, y offset                | Moves the mask horizontally and vertically by                                                                                       |                                                                                         |
| Mask erosion (-) / dilation (+) | Enlarge or reduce the detected mask.                                                                                                | [opencv example](https://docs.opencv.org/4.7.0/db/df6/tutorial_erosion_dilatation.html) |
| Mask merge mode                 | `None`: Inpaint each mask<br/>`Merge`: Merge all masks and inpaint<br/>`Merge and Invert`: Merge all masks and Invert, then inpaint |                                                                                         |

Applied in this order: x, y offset → erosion/dilation → merge/invert.

#### Inpainting

Each option corresponds to a corresponding option on the inpaint tab. Therefore, please refer to the inpaint tab for usage details on how to use each option.

## ControlNet Inpainting

You can use the ControlNet extension if you have ControlNet installed and ControlNet models.

Support `inpaint, scribble, lineart, openpose, tile` controlnet models. Once you choose a model, the preprocessor is set automatically. It works separately from the model set by the Controlnet extension.

## Advanced Options

API request example: [wiki/API](https://github.com/Bing-su/adetailer/wiki/API)

`ui-config.json` entries: [wiki/ui-config.json](https://github.com/Bing-su/adetailer/wiki/ui-config.json)

`[SEP], [SKIP]` tokens: [wiki/Advanced](https://github.com/Bing-su/adetailer/wiki/Advanced)

## Media

- 🎥 [どこよりも詳しいAfter Detailer (adetailer)の使い方① 【Stable Diffusion】](https://youtu.be/sF3POwPUWCE)
- 🎥 [どこよりも詳しいAfter Detailer (adetailer)の使い方② 【Stable Diffusion】](https://youtu.be/urNISRdbIEg)

## Model

| Model                 | Target                | mAP 50                        | mAP 50-95                     |
| --------------------- | --------------------- | ----------------------------- | ----------------------------- |
| face_yolov8n.pt       | 2D / realistic face   | 0.660                         | 0.366                         |
| face_yolov8s.pt       | 2D / realistic face   | 0.713                         | 0.404                         |
| hand_yolov8n.pt       | 2D / realistic hand   | 0.767                         | 0.505                         |
| person_yolov8n-seg.pt | 2D / realistic person | 0.782 (bbox)<br/>0.761 (mask) | 0.555 (bbox)<br/>0.460 (mask) |
| person_yolov8s-seg.pt | 2D / realistic person | 0.824 (bbox)<br/>0.809 (mask) | 0.605 (bbox)<br/>0.508 (mask) |
| mediapipe_face_full   | realistic face        | -                             | -                             |
| mediapipe_face_short  | realistic face        | -                             | -                             |
| mediapipe_face_mesh   | realistic face        | -                             | -                             |

The yolo models can be found on huggingface [Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer).

### Additional Model

Put your [ultralytics](https://github.com/ultralytics/ultralytics) yolo model in `webui/models/adetailer`. The model name should end with `.pt` or `.pth`.

It must be a bbox detection or segment model and use all label.

## How it works

ADetailer works in three simple steps.

1. Create an image.
2. Detect object with a detection model and create a mask image.
3. Inpaint using the image from 1 and the mask from 2.
