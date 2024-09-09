title:: func (*InvalidUnmarshalError) Error ¶ (highlights)
author:: [[go.dev]]
full-title:: "func (*InvalidUnmarshalError) Error ¶"
category:: #articles
url:: https://pkg.go.dev/encoding/json
summary:: The text describes various functions for handling JSON data in Go, including encoding, decoding, and formatting JSON. Key functions include `Marshal` for converting Go values to JSON and `Unmarshal` for parsing JSON back into Go values. Additional functions like `Compact`, `HTMLEscape`, and `Indent` help format JSON for specific uses, such as embedding in HTML.
![](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- Handling of anonymous struct fields is new in Go 1.1. Prior to Go 1.1, anonymous struct fields were ignored. To force ignoring of an anonymous struct field in both current and earlier versions, give the field a JSON tag of "-". ([View Highlight](https://read.readwise.io/read/01j677aq3mbkxe5mtjsrj6pwb2))
	- Map values encode as JSON objects. The map's key type must either be a string, an integer type, or implement [encoding.TextMarshaler](https://pkg.go.dev/encoding#TextMarshaler). The map keys are sorted and used as JSON object keys by applying the following rules, subject to the UTF-8 coercion described for string values above:
	  
	  •   keys of any string type are used directly
	  •   keys that implement [encoding.TextMarshaler](https://pkg.go.dev/encoding#TextMarshaler) are marshaled
	  •   integer keys are converted to strings ([View Highlight](https://read.readwise.io/read/01j677bxd0g1wwx33xt6c34ak4))
		- 💡: the map keys are sorted