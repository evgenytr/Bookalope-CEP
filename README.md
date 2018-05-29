# Bookalope-CEP

## Introduction

[Adobe CEP](https://github.com/Adobe-CEP), or Common Extensibility Platform, provides a framework to extend Adobe applications like [InDesign](http://www.adobe.com/products/indesign.html) with custom user interfaces and functionality. There is a wealth of useful extensions available on [Adobe’s Add-On Marketplace](https://creative.adobe.com/addons).

This repository contains the complete source code for [Bookalope](https://bookalope.net/)’s InDesign extension. The extension uses the [Bookalope REST API](https://github.com/jenstroeger/Bookalope/blob/master/API.md) and its [Javascript wrapper](https://github.com/jenstroeger/Bookalope/blob/master/clients/javascript/bookalope.js) to integrate the Bookalope toolset into InDesign. For more information on Bookalope and its services, please refer to the [Bookalope](https://bookalope.net) website.

## Requirements

The extension requires and supports Adobe InDesign versions 11–13. However, the extension may run on older versions that provide CEP support, but that hasn’t been tested.

## Coding

Read a general introduction to writing an Adobe CEP extension [here](http://www.adobe.com/devnet/creativesuite/articles/a-short-guide-to-HTML5-extensions.html). For more information on the Bookalope API please refer to its [documentation on Github](https://github.com/jenstroeger/Bookalope).

InDesign does not load an extension unless the extension has been cryptographically signed. Therefore, in order to run the Bookalope extension from source, you need to switch InDesign to _PlayerDebugMode_:

 - On Mac, open the file `~/Library/Preferences/com.adobe.CSXS.8.plist` and add a row with key `PlayerDebugMode` of type String and value `1`. You can edit `.plist` files using [Xcode](https://developer.apple.com/xcode/), [PList Edit](https://www.fatcatsoftware.com/plisteditpro/) or other applications suited to edit [Property Lists](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/PropertyLists/Introduction/Introduction.html).
 - On Windows, using its [registry editor](https://support.microsoft.com/en-au/help/4027573/windows-open-registry-editor-in-windows-10) open the registry key `HKEY_CURRENT_USER/Software/Adobe/CSXS.8` and add a key named `PlayerDebugMode` of type String and value `1`.

If you have multiple CSXS versions installed, add the `PlayerDebugMode` to all of them.

For InDesign to find and load the Bookalope extension, copy the entire `extensions/Bookalope` folder to:

 - On Mac: `~/Library/Application Support/Adobe/CEP/extensions`
 - On Windows: `%APPDATA%\Adobe\CEP\extensions`

Then start InDesign, and the extension should be available from the _Window → Extensions_ menu.

### Debugging

Debugging the ExtendScript side of the extension doesn’t really work. However, I found it helpful to use the [ExtendScript Toolkit](http://www.adobe.com/au/products/extendscript-toolkit.html), connect it to a running InDesign instance, and prototype the code there before using it for the Bookalope extension.

Debugging the Javascript side is easier. The file `extensions/Bookalope/.debug` configures a remote debugger which can connect to a running InDesign instance. Once the extension panel is open, navigate the [Chrome web browser](https://www.google.com/chrome/) (or alternatively, the [Adobe cef-client](https://github.com/Adobe-CEP/CEP-Resources)) to `localhost:8001` and enjoy…

## Building

To build the extension and generate an Adobe Zip Format Extension Package (`.zxp`) use Adobe’s [ZXPSignCmd tool](https://github.com/Adobe-CEP/CEP-Resources/tree/master/ZXPSignCMD). You also need a valid [PKCS 12](https://en.wikipedia.org/wiki/PKCS_12) certificate. If you don’t have one yet, you may generate one using e.g. [OpenSSL](https://www.ssl.com/how-to/create-a-pfx-p12-certificate-file-using-openssl/) or the ZXPSignCmd itself:

    > # ZXPSignCmd -selfSignedCert <countryCode> <stateOrProvince> <organization> <commonName> <password> <outputPath.p12> [options]
    > ZXPSignCmd -selfSignedCert AU QL Bookalope Bookalope MyPa55w0rd bookalope-cep-cert.p12

Once you’ve created the certificate, package and sign the extension itself:

    > # ZXPSignCmd -sign <inputDirectory> <outputZxp> <p12> <p12Password> [options]
    > ZXPSignCmd -sign ./extensions/Bookalope com.bookalope.zxp bookalope-cep-cert.p12 MyPa55w0rd -tsa https://timestamp.geotrust.com/tsa

Finding a [Trusted Timestamping Authority (TSA)](https://en.wikipedia.org/wiki/Trusted_timestamping) that works for ZXPSignCmd seems to be challenging at times, so take a look at [this list of possible options](http://www.davidebarranca.com/2017/04/html-panel-tips-24-fixing-zxp-timestamping-errors/).

## Installing

The easy way to install the packaged and signed extension is by using [Anastasiy’s Extension Manager](http://install.anastasiy.com/). The nerdy way is to use Adobe’s [ExManCmd tool](https://www.adobeexchange.com/resources/28):

    > ExManCmd --list all  # List all installed extensions
    > ExManCmd --install ./com.bookalope.zxp  # Install the Bookalope extension

Then start InDesign, and the extension should be available from the _Window → Extensions_ menu.

## Running

Once installed, open the extension panel:

<img src="https://bookalope.net/img/idsn-bookalope-upload.png" width="50%" alt="Bookalope InDesign: Upload panel">

To use the Bookalope web services, you need a Bookalope API key: you can find the key in the [user profile](https://bookflow.bookalope.net/profile) of your Bookalope account. Then select a book manuscript in Word format (or any other of the [supported document formats](https://github.com/jenstroeger/Bookalope/blob/master/API.md#get-formats)), fill in `Name` and `Author` of the book, and then click the `Upload and convert` button.

The extension uploads the document to the Bookalope server for analysis and conversion. This can take a little while. Once finished, the extension downloads the converted file, creates a new InDesign document, and places the downloaded content into the new InDesign document. The extension’s panel changes now to this:

<img src="https://bookalope.net/img/idsn-bookalope-update.png" width="50%" alt="Bookalope InDesign: Update panel">

From here, you can open the document on the Bookalope website to revisit and adjust the entire analysis and conversion flow (watch the video tutorials on [Youtube](https://www.youtube.com/channel/UCCxR_k6G06qEAj3IjZ9AcoQ)). You can also download the converted document in EPUB, MOBI, or print-ready PDF formats.

## Further Documentation

Writing an Adobe CEP extension requires documentation which is not always readily available. However, some useful resources are:

 - The Adobe CEP resources on Github is [here](https://github.com/Adobe-CEP/CEP-Resources);
 - The ExtendScript documentation is [here](http://www.indesignjs.de/extendscriptAPI/indesign-latest/);
 - Davide Barranca’s [blog](https://www.davidebarranca.com/category/code/) contains useful tips;
 - Adobe CEP extensions’s UI should be styled using [CEP Spectrum Boilerplate](https://github.com/Hennamann/CEP-Spectrum-Boilerplate) or [Topcoat](https://github.com/topcoat/topcoat).

In addition to the above, the [InDesign Scripting Forum](https://forums.adobe.com/community/indesign/indesign_scripting) might be useful, or the [#cep channel on Slack](https://adobedevs.slack.com/messages/C1FKLQ63F) is a playground to many experienced developers.