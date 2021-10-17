To create an OpenC2 Actuator Profile, create a profile folder,
copy `ap-template` into it, and follow the instructions in
https://github.com/oasis-tcs/openc2-usecases/tree/main/Actuator-Profile-Schemas.

To create an OpenC2 device/service that supports one or more actuator profiles,
copy the `lang` template and configure it for the desired profiles.

To test the Device, create four directories `Good-command`, `Bad-command`, `Good-response`, and `Bad-response`
in the device folder, populate them with test commands/responses, and run the test script.