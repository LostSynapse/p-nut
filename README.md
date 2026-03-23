This is a config for an esphome NUT stack intended for PoE esp32-p4 devices.

It has not been tested, as I have not yet received the hardware.

This config links to my personal fork (unmodified) of bullshit\esphome-components


Instructions for compile:

Create a secrets.yaml in the root of this folder with the following:

```
api_key: "generatedAPIkey"
ota_password: "changethispassword"
```

You can generate an API key with the following:

```
python3 -c "import base64, os; print(base64.b64encode(os.urandom(32)).decode())"
```

Run the following in the directory (esphome must be installed)

```
esphome compile p-nut_waveshare.yaml
```
or
```
esphome compile p-nut_m5stack.yaml
```