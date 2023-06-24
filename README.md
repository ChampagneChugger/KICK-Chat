# KICK Chat

Display Kick chat on your stream via browser source.

## How To

Add a Browser Source in your OBS and modify the width and height to your liking (I recommend only changing height). In the URL field paste in this:

### https://kick-chat-alpha.vercel.app/?username=USERNAME

Replace the USERNAME placeholder with your KICK username (it's not case sensitive).
That's it. If you wish to modify it a little bit, you could also pass optional parameters, such as: pictures, roles.

pictures - disable display of profile pictures

roles - disable display of user roles

### https://kick-chat-alpha.vercel.app/?username=USERNAME&pictures=0

You can stack parameters with &, such as this:

### https://kick-chat-alpha.vercel.app/?username=USERNAME&pictures=0&roles=0
