[vulnhub - easy/medium] bulldog: 1
----------------------------------

### enumeration
Since the machine's ip is already shown at its login page, I decide to check it on my browser to see if there's any useful information in it:
![bulldog 1](./images/bulldog-1.png)
But there isn't relevant info on the page (even after reading the source code) nor in the "Public Notice" link above the bulldog photo (except for text, that is, which could be used in CeWL if it comes to that). Running exiftoon on the photo also doesn't give me anything useful, just the cheeky nod:
```bash
Comment                         : Not a part of the VM, he's just cute :3 https://www.pexels.com/photo/white-and-brown-bulldog-on-brown-wood-planks-160748/
```

