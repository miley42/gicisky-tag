# About this fork

This fork is a quick and dirty solution for pushing images to a 2.1" BW Picksmart tag that has some differences in the protocol and image encoding.

The code version in this fork currently only supports black and white images, no red, no dithering.

With the original code, it got stuck at the first `Sending image part` because the tag didn't acknowledge it (didn't send a command to request the following part).
This is resolved by adding three 0x00 bytes at the end of the 0x02 image size command. (The web-based uploader at https://atc1441.github.io/ATC_GICISKY_Paper_Image_Upload.html
also uses this command).

The image encoding for this tag sets pixels in squares of four. So the first bit is the top left pixel of the squre, the second top right, the third bottom left,
the fourth bottom right. Using these squares, the image is written in vertical lines, starting in the bottom right corner.

Also, even though the screen has a resolution of 250x122, it requires image data with a resolution of 250x132, of which the top 10 pixels are cropped.

Example command: `$ gicisky-tag-writer --image example.png -v`, with example.png being a 250x132 black and white image file.

Original README follows:

# Gicisky Bluetooth ESL e-paper tag

![Tag](docs/tag.jpg)

This repository provides a `gicisky-tag-writer` script and a `gicisky_tag` Python library to write custom images to a Gicisky / PICKSMART electronic price tag (also called electronic shelf label, or ESL), provided that it's programmable via Bluetooth ESL. So far the project has been tested only on the model "2.1 inch EPA LCD 250x122 BWR". If you have a different device, feel free to open a PR to generalize the code.

This Python project uses [uv](https://github.com/astral-sh/uv) to manage all dependencies. To run the script from the repository folder:
```bash
git clone https://github.com/fpoli/gicisky-tag.git
cd gicisky-tag
uv sync
uv run gicisky-tag-writer --help
```

Alternatively, to install the script without cloning the repository, use [pipx](https://pypa.github.io/pipx/):
```bash
pipx install git+https://github.com/fpoli/gicisky-tag.git
gicisky-tag-writer --help
```

## Usage

```text
$ gicisky-tag-writer --help
usage: gicisky-tag-writer [-h] --image IMAGE [--address ADDRESS] [--dithering {none,floydsteinberg,combined}] [--debug-folder DEBUG_FOLDER]

Write an image to a Gicisky tag.

options:
  -h, --help            show this help message and exit
  --image IMAGE         Image to send.
  --address ADDRESS     Bluetooth address of the Gicisky tag to be updated. If not provided, the script will scan and use the first Gicisky tag that it can find.
  --dithering {none,floydsteinberg,combined}
                        Dithering method (default: none).
  --debug-folder DEBUG_FOLDER
                        Folder in which to save debug data.
```
## Documentation

Officially, to write to the tags, you need to [register an account](http://a.picksmart.cn:8082/index) and [download an app](http://www.picksmart.cn/index.php/page-22-11.html) on the Picksmart website. In my case, I used the APK [`ble-tag-english-app-release-v3.1.37.apk`](http://a.picksmart.cn:8088/picksmart/app/ble-tag-english-app-release-v3.1.32.apk).
I don't know why their app is not on the official app store, so install and use it at your own risk.
This project makes it possible to write custom images to the tags without using any proprietary service or app.

The Bluetooth ESL protocol to update the screen is described [here](https://zhuanlan.zhihu.com/p/633113543).
Independently, [`atc1441`](https://github.com/atc1441) reverse-engineered the protocol and published a JavaScript image uploader ([video](https://www.youtube.com/watch?v=Cp4gNXtlbGk), [repo](https://github.com/atc1441/ATC_GICISKY_ESL), [uploader](https://atc1441.github.io/ATC_GICISKY_Paper_Image_Upload.html)) that in my case managed to write something on the screen, although the result was gibberish because the base-64 encoded image data provided as default in the uploader is for a different screen model.
Modifying the image data is not trivial, because it uses an undocumented compression format.
[`Cabalist`](https://github.com/Cabalist) partially reverse-engineered the image format ([his notes](https://github.com/Cabalist/gicisky_image_notes)) revealing a form of run-length encoding. Later, it was discovered that the compression format is [QuickLZ](https://github.com/RT-Thread-packages/quicklz), compressing each line independently at level 1 in streaming mode.
For simplicity, this project transmits uncompressed images. In my tests, compressed images turned out to be often bigger than the uncompressed ones, probably due to the dithering that makes compression harder, and also because the streaming compression state is not reused across lines.

A copy of some of the material linked above is stored in the `docs` folder.

## Details of tag model 2.1" EPA LCD 250x122 BWR

* Bluetooth name: "PICKSMART" while it's powering up, then "NEMRxxyyzzkk"
* Bluetooth address: `FF:FF:xx:yy:zz:kk`
* Brand: Gicisky / PICKSMART
* Batteries: 2 replaceable CR2450 3V

The `xxyyzzkk` above corresponds to the number encoded by the barcode on the right of the screen.

## License

License of the project, except for the content of the `docs/` folder:

Copyright (C) 2023  Federico Poli

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
