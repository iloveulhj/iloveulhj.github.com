---
layout: post
title:  "[JAVA] String API - charset"
date:   2015-06-28 00:00:00
categories: posts java
---

# Java String API - charset

#### Constrouctor

| **API** | **Description** |
| String(byte[] bytes) | Constructs a new String by decoding the specified array of bytes using **the platform's default charset.** |
| String(byte[] bytes, Charset charset) | Constructs a new String by decoding the specified array of bytes using **the specified charset.** |

**String API source:**  
(Read more: grep code - [String.class](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/lang/String.java#String.%3Cinit%3E%28byte%5B%5D%2Cjava.nio.charset.Charset%29), [StringCoding.class](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/lang/StringCoding.java#StringCoding.decode%28java.lang.String%2Cbyte%5B%5D%2Cint%2Cint%29))
{% highlight java %}
/* String.class */
public String(byte bytes[], Charset charset) {
	this(bytes, 0, bytes.length, charset);
}

public String(byte bytes[], int offset, int length) {
	checkBounds(bytes, offset, length);
	this.value = StringCoding.decode(bytes, offset, length);
}

/* StringCoding.class */
static char[] decode(byte[] ba, int off, int len) {
	String csn = Charset.defaultCharset().name();
	try {
		// use charset name decode() variant which provides caching.
		return decode(csn, ba, off, len);
	} catch (UnsupportedEncodingException x) {
		warnUnsupportedCharset(csn);
	}
	try {
		return decode("ISO-8859-1", ba, off, len);
	} catch (UnsupportedEncodingException x) {
		// If this code is hit during VM initialization, MessageUtils is
		// the only way we will be able to get any kind of error message.
		MessageUtils.err("ISO-8859-1 charset not available: "
						 + x.toString());
		// If we can not find ISO-8859-1 (a required encoding) then things
		// are seriously wrong with the installation.
		System.exit(1);
		return null;
	}
}

static char[] decode(String charsetName, byte[] ba, int off, int len) throws UnsupportedEncodingException {
	StringDecoder sd = deref(decoder);
	String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
	if ((sd == null) || !(csn.equals(sd.requestedCharsetName())
						  || csn.equals(sd.charsetName()))) {
		sd = null;
		try {
			Charset cs = lookupCharset(csn);
			if (cs != null)
				sd = new StringDecoder(cs, csn);
		} catch (IllegalCharsetNameException x) {}
		if (sd == null)
			throw new UnsupportedEncodingException(csn);
		set(decoder, sd);
	}
	return sd.decode(ba, off, len);
}

char[] decode(byte[] ba, int off, int len) {
	int en = scale(len, cd.maxCharsPerByte());
	char[] ca = new char[en];
	if (len == 0)
		return ca;
	if (cd instanceof ArrayDecoder) {
		int clen = ((ArrayDecoder)cd).decode(ba, off, len, ca);
		return safeTrim(ca, clen, cs, isTrusted);
	} else {
		cd.reset();
		ByteBuffer bb = ByteBuffer.wrap(ba, off, len);
		CharBuffer cb = CharBuffer.wrap(ca);
		try {
			CoderResult cr = cd.decode(bb, cb, true);
			if (!cr.isUnderflow())
				cr.throwException();
			cr = cd.flush(cb);
			if (!cr.isUnderflow())
				cr.throwException();
		} catch (CharacterCodingException x) {
			// Substitution is always enabled,
			// so this shouldn't happen
			throw new Error(x);
		}
		return safeTrim(ca, cb.position(), cs, isTrusted);
	}
}
{% endhighlight %}

**The platform's default charset:**    
(Read more: [Javarevisited - How to get and set default Character encoding or Charset in Java](http://javarevisited.blogspot.com/2012/01/get-set-default-character-encoding.html#ixzz3eM6L9518)) 

* Default Character encoding in Java or charset is the character encoding used by JVM to convert bytes into Strings or characters when you don't define java system property "file.encoding". Java gets character encoding by calling System.getProperty("file.encoding","UTF-8") at the time of JVM start-up  
* ways to get Default character encoding in Java
	* System.getProperty("file.encoding"), JVM started with -Dfile.encoding property 
	* Charset.defaultCharset()
	* InputStreamReader.getEncoding()

---

#### Method

| **API** | **Description** |
| byte[]	getBytes() | Encodes this String into a sequence of bytes using **the platform's default charset**, storing the result into a new byte array. |
| byte[]	getBytes(Charset charset) | Encodes this String into a sequence of bytes using **the given charset**, storing the result into a new byte array. |
| byte[]	getBytes(String charsetName) | Encodes this String into a sequence of bytes using **the named charset**, storing the result into a new byte array. |

**String API source:**  
(Read more: grep code - [String.class](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/lang/String.java#String.getBytes%28%29), [StringCoding.class](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/lang/StringCoding.java#StringCoding.encode%28char%5B%5D%2Cint%2Cint%29))
{% highlight java %}
/* String.class */
public byte[] getBytes() {
	return StringCoding.encode(value, 0, value.length);
}

public byte[] getBytes(String charsetName) throws UnsupportedEncodingException {
	if (charsetName == null) throw new NullPointerException();
	return StringCoding.encode(charsetName, value, 0, value.length);
}

public byte[] getBytes(Charset charset) {
	if (charset == null) throw new NullPointerException();
	return StringCoding.encode(charset, value, 0, value.length);
}

/* StringCoding.class */
static byte[] encode(String charsetName, char[] ca, int off, int len)
	throws UnsupportedEncodingException
{
	StringEncoder se = deref(encoder);
	String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
	if ((se == null) || !(csn.equals(se.requestedCharsetName())
						  || csn.equals(se.charsetName()))) {
		se = null;
		try {
			Charset cs = lookupCharset(csn);
			if (cs != null)
				se = new StringEncoder(cs, csn);
		} catch (IllegalCharsetNameException x) {}
		if (se == null)
			throw new UnsupportedEncodingException (csn);
		set(encoder, se);
	}
	return se.encode(ca, off, len);
}

byte[] encode(char[] ca, int off, int len) {
	int en = scale(len, ce.maxBytesPerChar());
	byte[] ba = new byte[en];
	if (len == 0)
		return ba;
	if (ce instanceof ArrayEncoder) {
		int blen = ((ArrayEncoder)ce).encode(ca, off, len, ba);
		return safeTrim(ba, blen, cs, isTrusted);
	} else {
		ce.reset();
		ByteBuffer bb = ByteBuffer.wrap(ba);
		CharBuffer cb = CharBuffer.wrap(ca, off, len);
		try {
			CoderResult cr = ce.encode(cb, bb, true);
			if (!cr.isUnderflow())
				cr.throwException();
			cr = ce.flush(bb);
			if (!cr.isUnderflow())
				cr.throwException();
		} catch (CharacterCodingException x) {
			// Substitution is always enabled,
			// so this shouldn't happen
			throw new Error(x);
		}
		return safeTrim(ba, bb.position(), cs, isTrusted);
	}
}
{% endhighlight %}

--- 

출처:

* [Oracle JAVA7 String API Document](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html)
* [Javarevisited - How to get and set default Character encoding or Charset in Java](http://javarevisited.blogspot.kr/2012/01/get-set-default-character-encoding.html)
* [Grep Code](http://grepcode.com/)