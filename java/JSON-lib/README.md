There are two *different* examples here.
The first uses `json-lib` (the "standard" JSON library), the second uses `google-gson`
(simpler, developed by Google, **probably what you you want**).
The code is [on Github](https://github.com/coolaj86/json-examples/tree/master/java).

### Running the Example JSON Code

This example depends on the required jars being in `./lib`.
Necessary dependencies are:

  - JSON-Lib ([Project Page](http://json-lib.sourceforge.net/))
    - [commons-beanutils-1.8.x](http://mirrors.axint.net/apache//commons/beanutils/binaries/commons-beanutils-1.8.3-bin.tar.gz)
    - [commons-collections-3.2.x](http://apache.mirrors.pelicantech.com//commons/collections/binaries/commons-collections-3.2.1-bin.tar.gz)
    - [commons-lang-2.5+](http://apache.mirrors.pair.com//commons/lang/binaries/commons-lang3-3.1-bin.tar.gz)
    - [commons-logging-1.1.1](http://apache.mirrors.pair.com//commons/logging/binaries/commons-logging-1.1.1-bin.tar.gz)
    - [ezmorph-1.0.6](http://sourceforge.net/settings/mirror_choices?projectname=ezmorph&filename=ezmorph/ezmorph-1.0.6/ezmorph-1.0.6.jar)
    - [json-lib-2.4-jdk15.jar](http://sourceforge.net/settings/mirror_choices?projectname=json-lib&filename=json-lib/json-lib-2.4/json-lib-2.4-jdk15.jar)

  - Google GSON ([Project Page](http://code.google.com/p/google-gson/)) (**best**)
    - gson-2.1.jar

#### Gotta download and unpack 'em all:

    mkdir lib
    pushd lib
    wget \
      'http://mirrors.axint.net/apache//commons/beanutils/binaries/commons-beanutils-1.8.3-bin.tar.gz' \
      'http://apache.mirrors.pelicantech.com//commons/collections/binaries/commons-collections-3.2.1-bin.tar.gz' \
      'http://apache.mirrors.pair.com//commons/lang/binaries/commons-lang3-3.1-bin.tar.gz' \
      'http://apache.mirrors.pair.com//commons/logging/binaries/commons-logging-1.1.1-bin.tar.gz' \
      'http://iweb.dl.sourceforge.net/project/ezmorph/ezmorph/ezmorph-1.0.6/ezmorph-1.0.6.jar' \
      'http://iweb.dl.sourceforge.net/project/json-lib/json-lib/json-lib-2.4/json-lib-2.4-jdk15.jar' \
      'http://google-gson.googlecode.com/files/google-gson-2.1-release.zip' \

    mv *ezmorph-1.0.6.jar ezmorph-1.0.6.jar
    mv *json-lib-2.4-jdk15.jar json-lib-2.4-jdk15.jar

    ls | grep tar | while read F; do tar xf $F; done
    ls | grep zip | while read F; do unzip $F; done

    mv ./*/*.jar ./
    popd

#### Compile and Run

    cmake .
    make
    java -cp .:lib/* example 192.168.254.254 # ip of sensor

### Code Walkthrough ###

There are four major functions:

- `getSettings`: make HTTP request to spotter and get the settings
- `setSettings`: make HTTP request to spotter to set settings
- `handleSettingsJsonLib`: JSON example using Json-Lib
- `handleSettingsGson`: JSON example using Gson

#### getSettings ####

Start off by creating a URL object,
being sure to tack on the desired resource want in the constructor:

    URL tUrl = new URL(url + RESOURCE);

Open the connection:

    URLConnection req = tUrl.openConnection();

Get the data from the response stream:

    BufferedReader in = new BufferedReader(new InputStreamReader(req.getInputStream()));

    StringBuilder sb = new StringBuilder();
    String input;
    while ((input = in.readLine()) != null) {
        sb.append(input);
    }

That's it! The `input` string now contains the response from the json app;
we use a JSON library below to manipulate it.

#### setSettings ####

This is much the same as the `getSettings` example, except we need to make sure the resource is `/settings` and that we set some headers.
To start off, we get the bytes from our new settings:

    // we use UTF-8 because that's what the spotters use internally
    byte[] bytes = settings.getBytes("UTF-8");

Then we set up our connection:

    URL tUrl = new URL(url + RESOURCE + "/settings");
    HttpURLConnection req = (HttpURLConnection)tUrl.openConnection();
    req.setDoOutput(true);

    req.setRequestMethod("GET");
    req.setRequestProperty("Content-Type", "application/json");
    req.setFixedLengthStreamingMode(bytes.length);

You'll notice a few differences here.
First of all, we use an `HttpURLConnection` instead of the `URLConnection`.
In `getSettings`, we could have used a `HttpURLConnection` object instead,
but we didn't need the features, so it was simpler to not typecast it.

Here, however, we need to be able to set the HTTP method and a few headers,
so we use the real structure.

Next, we send our data up to the Spotter:

    DataOutputStream out = new DataOutputStream(req.getOutputStream());
    out.write(bytes, 0, bytes.length);
    out.flush();
    out.close();

And then get the response as before:

    BufferedReader in = new BufferedReader(new InputStreamReader(req.getInputStream()));

    StringBuilder sb = new StringBuilder();
    String input;
    while ((input = in.readLine()) != null) {
        sb.append(input);
    }

#### getSettingsGzip ####

This sets the "Accept-Encoding" header on the http request:

    req.setRequestProperty("Accept-Encoding", "gzip;q=1.0");

On the response, we need to check the "Content-Encoding" header.
If gzip is present, then set a boolean so we know how to interpret the response:

    // get the response headers
    Map<String, List<String>> headers = req.getHeaderFields();

    for (String key : headers.keySet()) {
        if (key != null && key.equalsIgnoreCase("Content-Encoding")) {
            for (String val : headers.get(key)) {
                if (val.equalsIgnoreCase("gzip")) {
                    gzipped = true;
                    break;
                }
            }
            break;
        }
    }

If it is encoded as gzip, then create a new `GZIPInputStream` and
let it handle decompressing the response:

    GZIPInputStream in = new GZIPInputStream(req.getInputStream());

Read it out however you like.
GZIPInputStream is in the standard library,
so it supports the common stream operations.

If it is not encoded as gzip, then handle it the same way `getSettings` does.

#### handleSettingsJsonLib ####

Json-lib has a lot of features, but we'll only use a small subset for this example.
It is one of the more popular JSON libraries,
probably because it is based on the
[reference implementation](http://www.json.org/java) by Douglas Crockford.

To start off, we'll use `JSONSerializer` to parse our settings into a `JSONObject`:

    JSONObject obj = (JSONObject)JSONSerializer.toJSON(settings);

Then get the `result` property:

    JSONObject res = obj.getJSONObject("result");

Then we'll grab the altitude property and do some work with it:

    double alt = res.getDouble("altitude");

When we're done, just put it back:

    res.put("altitude", alt);

To grab the data as a string, just call it's `toString` method.
That's it!

#### handleSettingsGson ####

Google-Gson is Google's implementation of a JSON parser.
This is my preferred library for parsing JSON in Java mostly because it has no dependencies.

Gson has a way to serialize JSON directly to a Java Object (using a class definition), but we'll just use a simple iterative implementation for this example:

First, create the parser and parse the data into a JsonObject:

    JsonParser parser = new JsonParser();
    JsonObject obj = parser.parse(settings).getAsJsonObject();

Then pull the `result` object out of the data:

    JsonObject result = obj.getAsJsonObject("result");

Once we have this, we grab the altitude property and do some work with it:

    float alt = result.get("altitude").getAsFloat();

And when we're done, we'll put it back in:

    result.addProperty("altitude", alt);

To grab the data as a string, just call its `toString` method.
That's it, we're done!

