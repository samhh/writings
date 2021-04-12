---
slug: "aerc-gpg-patch"
date: "2021-04-12"
title: "Patching aerc to directly access keys from GPG's keyring"
---

[aerc](https://sr.ht/~sircmpwn/aerc/) is really nice, but it doesn't yet support PGP all that well. [What support does exist](https://lists.sr.ht/~sircmpwn/aerc/%3CC11JC39B2EUP.9Y18EIDU7UMO%40homura%3E) is user-unfriendly and, worse, as far as I can tell (?), highly insecure.

There are [some issues](https://todo.sr.ht/~sircmpwn/aerc2?search=label%3A%22pgp%22) open on sourcehut to improve the state of PGP support. In the meantime, what is one to do if one wishes to maintain the integrity of their keyring? This is open source, so patch in what one wants of course!

Here's a Git patch that can be applied to master at time of writing:

```git
diff --git a/lib/messageview.go b/lib/messageview.go
index 08ea92f..f2b730f 100644
--- a/lib/messageview.go
+++ b/lib/messageview.go
@@ -2,6 +2,8 @@ package lib

 import (
 	"bytes"
+	"os"
+	"os/exec"
 	"io"
 	"io/ioutil"

@@ -67,6 +69,23 @@ func NewMessageStoreView(messageInfo *models.MessageInfo,
 	if usePGP(messageInfo.BodyStructure) {
 		store.FetchFull([]uint32{messageInfo.Uid}, func(fm *types.FullMessage) {
 			reader := fm.Content.Reader
+
+			// If no keyring yet, hook in here. Only downside is
+			// needing to enter the password twice.
+			if Keyring == nil {
+				privcmd := exec.Command("sh", "-c", "gpg --export-secret-keys hello@samhh.com")
+				privcmd.Stdin = os.Stdin
+				priv, err := privcmd.Output()
+				if err != nil {
+					return
+				}
+
+				Keyring, err = openpgp.ReadKeyRing(bytes.NewReader(priv))
+				if err != nil {
+					panic(err)
+				}
+			}
+
 			pgpReader, err := pgpmail.Read(reader, Keyring, decryptKeys, nil)
 			if err != nil {
 				cb(nil, err)
```

It's very hacky, hence me not formally contributing this upstream. You will of course want to change the ID of the relevant secret key in the call to `exec.Command`.

Upon accessing a PGP-encrypted message, it checks if a keyring internal to aerc exists. If not - and it won't if you haven't done what we're seeking to avoid - it will make a call directly to GPG to fetch the relevant secret key. The only downside of this approach, as noted in the patch, is that without further tinkering this will require you to twice enter your password; first for GPG, and then again for aerc, because the secret key doesn't go straight to decrypting the message but instead into aerc's internal keyring in memory.

To apply this patch you'll need to build aerc locally. Credit to the project here: without any experience with Go, I was able to get up and building very quickly. I may be forgetting a one-time initial command, but at this point it's just a case of running `go run .` to test it, and then `go build . && sudo make install` to install it (you could presumably manually move the binary into your `$PATH` without the need to run foreign code with root privileges).

