# `GWT JSON Overlays` #


# gwt\_overlay #

Generate code via **protostuff-compiler**.

Enable the option **dev\_mode** during active development on hosted mode.

See the [compiler options](CompilerOptions#gwt_overlay.md) for more details.

`foo.proto`
```
package foo;

option optimize_for = LITE_RUNTIME;
option java_package = "com.example.foo";

message Person {
  required int32 id = 1;
  optional string name = 2;
  optional string motto = 3 [default="When the cat is away, the mouse is alone!"];
  enum Gender {
    MALE = 1;
    FEMALE = 2;
  }
  optional Gender gender = 4;
}
```

Enabled options:
  * use\_global\_json
  * generate\_helper\_methods

The complete output:

```

package com.example.foo;

import com.google.gwt.core.client.*;

public final class Person extends JavaScriptObject  {

    public static final class Gender extends JavaScriptObject {
            
        public static final Gender MALE = create(1);
        public static final Gender FEMALE = create(2);
        
        private static native Gender create(int number) /*-{
            return number;
        }-*/;
        
        protected Gender() {}
        
        public native int getNumber() /*-{
            return this;
        }-*/;
        
    }

    
    /**
     * Creates a new Person instance 
     *
     * @return new Person instance
     */
    public static native Person create() /*-{
        return {
                        
        };
    }-*/;

    /**
     * Creates a new JsArray<Person> instance 
     *
     * @return new JsArray<Person> instance
     */
    public static native JsArray<Person> createArray() /*-{
        return [];
    }-*/;

    /**
     * Gets a Person (casting) from a JavaScriptObject
     *
     * @param JavaScriptObject to cast
     * @return Person
     */
    public static native Person get(JavaScriptObject jso) /*-{
        return jso;
    }-*/;

    /**
     * Gets a JsArray<Person> (casting) from a JavaScriptObject
     *
     * @param JavaScriptObject to cast
     * @return JsArray<Person> 
     */
    public static native JsArray<Person> getArray(JavaScriptObject jso) /*-{
        return jso;
    }-*/;

    /**
     * Parses a Person from a json string
     *
     * @param json string to be parsed/evaluated
     * @return Person 
     */
    public static native Person parse(String json) /*-{
        return $wnd.JSON.parse(json);
    }-*/;

    /**
     * Parses a JsArray<Person> from a json string
     *
     * @param json string to be parsed/evaluated
     * @return JsArray<Person> 
     */
    public static native JsArray<Person> parseArray(String json) /*-{
        return $wnd.JSON.parse(json);
    }-*/;
    
    /**
     * Serializes a json object to a json string.
     *
     * @param Person the object to serialize
     * @return String the serialized json string
     */
    public static native String stringify(Person obj) /*-{
        return $wnd.JSON.stringify(obj);
    }-*/;
    
    public static native boolean isInitialized(Person obj) /*-{
        return obj["id"] != null;
    }-*/;

    protected Person() {}

    // getters and setters

    // id

    public native int getId() /*-{
        return this["id"] || 0;
    }-*/;

    public native void setId(int id) /*-{
        this["id"] = id;
    }-*/;

    public native void clearId() /*-{
        delete this["id"];
    }-*/;

    public native boolean hasId() /*-{
        return this["id"] != null;
    }-*/;

    // name

    public native String getName() /*-{
        return this["name"] || "";
    }-*/;

    public native void setName(String name) /*-{
        this["name"] = name;
    }-*/;

    public native void clearName() /*-{
        delete this["name"];
    }-*/;

    public native boolean hasName() /*-{
        return this["name"] != null;
    }-*/;

    // motto

    public native String getMotto() /*-{
        var v = this["motto"];
        return v == null ? "When the cat is away, the mouse is alone!" : v;
    }-*/;

    public native void setMotto(String motto) /*-{
        this["motto"] = motto;
    }-*/;

    public native void clearMotto() /*-{
        delete this["motto"];
    }-*/;

    public native boolean hasMotto() /*-{
        return this["motto"] != null;
    }-*/;

    // gender

    public native Gender getGender() /*-{
        var v = this["gender"];
        return v == null ? 1 : v;
    }-*/;

    public native void setGender(Gender gender) /*-{
        this["gender"] = gender;
    }-*/;

    public native void clearGender() /*-{
        delete this["gender"];
    }-*/;

    public native boolean hasGender() /*-{
        return this["gender"] != null;
    }-*/;


}

```