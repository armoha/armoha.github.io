+++
title = "Nothing"

[taxonomies]
tags = ["thing", "no"]
+++

Nothing
==

thing ever thingie every

```go

func decryptMessage(filename string, decrypt bool) *gmime.Envelope {
	if !decrypt {
		return nil
	}
	if stream, err := os.Open(filename); err != nil {
		panic(err)
	} else {
		if fh, err := gpgme.Decrypt(stream); err != nil {
			log.Printf("decryptMessage error %v", err)
		} else {
			defer fh.Close()
			if data, err := ioutil.ReadAll(bufio.NewReader(fh)); err != nil {
				panic(err)
			} else {
				if envelope, err := gmime.Parse(string(data)); err != nil {
					panic(err)
				} else {
					return envelope
				}
			}
		}
	}
	return nil
}
```
