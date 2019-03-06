# Register for HackTheBox

1. Naviagate to https://www.hackthebox.eu/invite
1. Open developer tools
1. Open `/js/inviteapi.min.js`
    * The `makeInviteCode` function seems promising
1. Go to the console tab and run `makeInviteCode()`
1. Decode the Base 64 string
1. Make a POST request to https://www.hackthebox.eu/api/invite/generate
    * `curl -XPOST https://www.hackthebox.eu/api/invite/generate`
1. Decode the Base 64 string and use the code to register
