## Card

OwnTracks typically displays the [TID](tid.md) of a [friend](friends.md) on the map, but you can associate an address book entry to that friend on iOS in order to see a friendly face (if you have you friend's photo in your device's address book) and/or a friendly name.

![TID on iOS](images/b-ipad-TID-map.png)

We developed a new feature we call a _card_ which you can use when in both _MQTT mode_ and _HTTP mode_. A card is a retained message which contains a [JSON payload](../tech/json.md) which, in absence of an address-book association, will be used to populate your friend on your map. The payload contains a full name (hopefully one you recognize), and an avatar -- a small image. If a card exists it will be used, but you can override its use in OwnTracks by associating your friend with an address book entry of your own device.

```json
{
  "_type": "card",
  "name": "Jane Jolie",
  "face": "iV1CFEVkMhmCIKBUKh3 ... ghAAAAABJRU5ErkJggg=="
}
```

### Creating a card

Cards can be created with shell scripts or with a webapp.

#### Shell Script
We provide several utilities for creating a _card_ in the [Recorder's repository](https://github.com/owntracks/recorder/tree/master/contrib/faces):

* If you have an image file you want to use, use `image2card.sh`, passing _image-filename_ and _fullname_.
* If you know a user has a Github profile with a name and an avatar, use `github2card.py` which takes a Github username as argument.
* If you know a user has a Gravatar, use `gravatar2card.sh`, passing _email_ and _fullname_.
* Our [quicksetup](../guide/quicksetup.md) code contains an [example which uses `jo(1)`](https://github.com/owntracks/quicksetup/blob/main/files/mosquitto/janecardpub.sh).

These utilities create a _card_ on standard output, and you typically then publish the result as a retained message to your MQTT broker:

```
./github2card.py defunkt > my-card.json
mosquitto_pub -t owntracks/jjolie/phone/info -f my-card.json -r
```

Note the topic branch ending in `info` and note the use of the retain flag (`-r`).

#### Webapp

[This is a webapp][oc-demo] to create and edit [OwnTracks cards](../features/card.md).
It can be either used to just create the JSON representation of the card, which has to be published to the MQTT broker manually. Or it can directly publish the card to an MQTT broker connected via websockets.
Just head over to the [demo][oc-demo] and create a card. Configure your MQTT broker by clicking the connection state.
The source code can be found on [Github][oc-code].

[oc-demo]: https://avanc.github.io/owntracks-cards/
[oc-code]: https://github.com/avanc/owntracks-cards

### Generating the face image

We recommend formatting the face image as a 192x192 pixel image, encoded either as a JPEG or as a PNG. It is possible to use larger or smaller images, but 192x192 provides a good balance between image fidelity and data usage. Remember that this image will be transmitted many times over cellular data to all phones that use the app, so using very large images can be very costly.

You should use JPEG encoding when the image depicts something complex, such as a photography of a real face. You should use PNG encoding when the image only contains simple flat colors, such as icons or drawings.

We also recommend compressing the image to further save on data usage. A good app for this is [Squoosh](https://squoosh.app/), but be warned that [it uses Google trackers](https://github.com/GoogleChromeLabs/squoosh#privacy). For JPEG, the MozJPEG encoder with default settings is usually very good, and for PNG, the OxiPNG encoder with default settings is usually very good.

### Cards in Recorder in HTTP mode

Using HTTP mode in the OwnTracks Recorder will cause the Recorder to search for a friend's CARD in the following two paths, with the first found winning:

```
<STORAGEDIR>/cards/<user>/<user>-<device>.json
<STORAGEDIR>/cards/<user>/<user>.json
```

So, if `"jane"` is a friend with a device `"s8"`, the Recorder will load Jane's JSON card from the path `<STORAGEDIR>/cards/jane/jane-s8.json.` If Jane's device-specific card doesn't exist, her card is loaded from `<STORAGEDIR>/cards/jane.json` and silently ignored if that doesn't exist. (Note that as usual in Recorder, usernames and device names are lowercased.)

In `contrib/faces/` of the Recorder distribution there are some small utilities which can help create CARDs. Please make sure to verify that the `.json` file which you place into the directory is readable by the Recorder and is valid JSON. (You can test that with `jq . < file.json` or `python -mjson.tool file.json`.)

