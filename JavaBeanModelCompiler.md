# Flexible and inheritable schema generation #

**java\_bean\_model** compiler is designed to give you a lot of flexibility for generated schemas through out
polymorphism and extended use of annotations. You can subclass result schema and change the way fields are
written or read, change which fields should be written and which should not, how instances of deserialized
objects should be generated.

This compiler is based on basic **java\_bean** compiler code. The main purpose is to generate schemas
for described messages to be compatible to your models. It implies you already have your models in your
project and you don't want to change them much.

_Note. Models it self can be generated too if required. But a lot of specific features for that compiler
on models generation are not implemented yet. So the models you can get from it is much alike java\_bean
will give to you._

Second goal of this compiler is to give you some work around for handling inheritance in your models.
This also requires subclassing of generated schemas and explicitly specifying of how to behave with it.

# Features #

## Schema generation ##
  * inheritable schemas
  * overridable reads/writes
  * native type cast support on read/write int32
  * native byte[.md](.md) fields support
  * accessors (is/set/get), direct field access to public fields
  * renaming of field map
  * collection fields with base class types

## Model generation ##

  * immutable messages models (as option)
  * copy()
  * builder
  * toString()
  * native types for fields (support for existing annotation in schema @Byte, @Bytes, @Short, etc)

None of this model generation features are implemented yet.


# Usage #

  * Schema and Model classes will always be generated in separate files as separate classes.
  * Schema names will always be generated in form of: ` <message name><message suffix>Schema`. This is to separate names of generated models and schemas. Suffix can be defined following way (suffix term is used in postfix meaning) (options with higher priority overrides options with lower priority):
```
       // from highest priority to lowest 
       option msuffix = "Suffix"; // can only be provided inside of message body
       option java_model_suffix = "Suffix"; // can be provided inside of message body
       option <message name>.suffix = "Suffix for this certain message";  // in global namespace or in module properties
       option java_model_suffix = "Suffix"; // in global namespace, in module properties
```
  * Model names will always be generated in form of: ` <message package><message name><message suffix>`. Package is defined much alike schema Suffix. Package should be always end up with DOT.
```
       // from highest priority to lowest 
       option mpackage = "com.package."; // can only be provided inside of message body
       option java_model_package = "com.package."; // can be provided inside of message body
       option <message name>.package = "package for this certain message.";  // in global namespace or in module properties
       option java_model_package = "com.package."; // in global namespace, in module properties
```
  * if ` @Import(a="com.text.blah", b="org.text.blah", ...) ` is specified on a message, all values of params of this attribute will be included in an import section of the file.
  * if ` @Transient ` is specified on a message or enum block, it will be skipped for generation. Nothing will be generated for that.
  * ` option java_schema_field_accessors=true; ` can be used in module properties, or global namespace, or message body to specify that accessors methods should be used for accessing model fields on  reading/writing. _Note. Currently fields in a message will be used directly in one case only, when there is no of this option applied anywhere to certain message._
  * Pipe schema generation is untouched. I don't know is it useable currently in this version.
  * For a fields you can specify following attributes: ` @Byte, @Short, @Char ` to ask the compiler to provide needed casts on reads/writes. This is needed if you have native java values in your models. As you know there are no any protobuf field vals less then 32 bit per int. There is only one optimization for negative values. So, you can use this, to be compatible.
  * There is ` @Bytes ` attribute for your needs also. As you know default behaviour of protostuff is to provide you with ByteString array fields. This is not what you would like to get often. So, you can use this, if you have ` byte[] ` fields in your models.
  * There is ` @NoCheck ` attrubute. This is special for cases when you would like to omit nullity-check on a repeated field in mergeFrom body switch case. As you know, protostuff will check on deserialization of the repeated value whether it's target collection is not null or not, and will write a code for creating it if it is null. The implementation that is used for creation is ` ArrayList<T> `. This is not always what you want from it. So you can skip this check, if you need.
  * If you want to rename you field in field-map (to provide another names in json/xml) you can use ` @Field(alias = "...") ` attribute. Often you will want your serialized fields names to be different from what names your fields have in a model (java class).
  * There is also special attribute for repeated field called ` @Collection(name, type, checkFunction) `. It should be every argument of this attribute is optional, but it is useless without them. This attr will allow you to have custom name for your repeated field accessors (setName, getName i.e.), custom _superclass_ as generic argument to be used for collection initialization (and de/serialization) and special check function (that will return always false by default - yes, you should override it) that can help you to filter instances of your subclasses of collection superclass to be written with this specific field schema. This can be used to work around some inheritance issues, when you would like to have class hierarchy stored in one collection superclass.

# Sample: simple #

```
package game.data.model.proto;

option java_schema_field_accessors = true;
option java_model_package = "game.data.model.";
option java_model_suffix = "";

message Cowardice {
    required int32 defenderId = 1;

    required int32 attackerId = 2;

    @Byte
    required int32 score = 3;

    required int32 old = 4;

    @Byte
    required int32 lastResult = 5;
}

message CowardiceCacheEntry {
    option mpackage = "game.data.model.cache.";

    required int32 heroId = 1;

    repeated Cowardice cowardices  = 2;
}
```

# Sample: advanced - inheritance in collections #

Imagine you have a game project. In a compile package you
will have your items models and hero model. Weapons, Pets and
Potions are inherited from Item. There is HeroModel also that
is your model of hero and Hero final sub class also for business
logic.

compiler/Item.java:

```

package compiler;

public abstract class Item {
    private int id;
    private String name;

    protected Item() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

compiler/Pet.java

```

package compiler;

public class Pet extends Weapon {
    int hp;

    public Pet() {
    }

    public int getHp() {
        return hp;
    }

    public void setHp(int hp) {
        this.hp = hp;
    }
}

```

compiler/Weapon.java

```

package compiler;

public class Weapon extends Item {
    private int damage;

    public Weapon() {
    }

    public int getDamage() {
        return damage;
    }

    public void setDamage(int damage) {
        this.damage = damage;
    }
}

```

compiler/Potion.java

```
package compiler;

public class Potion extends Item {
    private int restore;

    public Potion() {
    }

    public int getRestore() {
        return restore;
    }

    public void setRestore(int restore) {
        this.restore = restore;
    }
}

```

compiler/HeroModel.java

```
package compiler;

import java.util.List;

public class HeroModel {
    private int id;
    private String name;
    private short hp;
    private short mp;

    private List<Item> items;

    public HeroModel() {}

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public short getHp() {
        return hp;
    }

    public void setHp(short hp) {
        this.hp = hp;
    }

    public short getMp() {
        return mp;
    }

    public void setMp(short mp) {
        this.mp = mp;
    }

    public List<Item> getItems() {
        return items;
    }

    public void setItems(List<Item> items) {
        this.items = items;
    }
}

```

compiler/Hero.java

```

package compiler;

public class Hero extends HeroModel {
    public Hero() {
    }

    // TODO: lots of logic here
}

```

Proto definitions will be as follows:

items.proto

```
package compiler.schema;

option java_schema_field_accessors = true;
option java_model_package = "compiler.";
option java_model_suffix = "";

@Transient
message Item {
    required int32 id           = 1;
    required string name        = 2;
}

message Pet {
    required int32 id           = 1;
    required string name        = 2;

    required int32 damage       = 3;
    required int32 hp           = 4;
}

message Potion {
    required int32 id           = 1;
    required string name        = 2;

    required int32 restore      = 3;
}

message Weapon {
    required int32 id           = 1;
    required string name        = 2;

    required int32 damage       = 3;
}
```

hero.proto

```
package compiler.schema;

import "items.proto";

option java_schema_field_accessors = true;

@Import(items = "compiler.*")
message Hero {
    option msuffix = "Model";

    required int32 id = 1;

    required string name = 2;

    @Short
    @Field(alias = "hitpoints")
    required int32 hp = 3;

    @Short
    @Field(alias = "manapoints")
    required int32 mp = 4;

    @Collection(name="items", type="Item", checkFunction="isPet")
    repeated Pet pets = 5;
    @Collection(name="items", type="Item", checkFunction="isWeapon")
    repeated Weapon weapons = 6;
    @Collection(name="items", type="Item", checkFunction="isPotion")
    repeated Potion potions = 7;
}
```

And after this, you will get in compiler/schema package 4 schema classes: PetSchema.java, PotionSchema.java, WeaponSchema.java, HeroModelSchema.java.

Items schemas will be much the same. PetSchema.java for example:

```
// Generated by http://code.google.com/p/protostuff/ ... DO NOT EDIT!
// Generated from proto

package compiler.schema;

import java.io.IOException;

import com.dyuproject.protostuff.Input;
import com.dyuproject.protostuff.Output;
import com.dyuproject.protostuff.Schema;
import com.dyuproject.protostuff.UninitializedMessageException;

public class PetSchema
       implements Schema<compiler.Pet> {


    static final compiler.Pet DEFAULT_INSTANCE = new compiler.Pet();
    static final Schema<compiler.Pet> SCHEMA = new PetSchema();

    public static compiler.Pet getDefaultInstance() { return DEFAULT_INSTANCE; }
    public static Schema<compiler.Pet> getSchema() { return SCHEMA; }

    public static final int FIELD_NONE = 0;
    public static final int FIELD_ID = 1;
    public static final int FIELD_NAME = 2;
    public static final int FIELD_DAMAGE = 3;
    public static final int FIELD_HP = 4;

    public PetSchema() {}


    public compiler.Pet newMessage() {
        return new compiler.Pet();
    }

    public Class<compiler.Pet> typeClass() {
        return compiler.Pet.class;
    }

    public String messageName() {
        return compiler.Pet.class.getSimpleName();
    }

    public String messageFullName() {
        return compiler.Pet.class.getName();
    }


    public boolean isInitialized(compiler.Pet message) {
        return true;
    }


    public void mergeFrom(Input input, compiler.Pet message) throws IOException {
        for (int fieldIx = input.readFieldNumber(this); fieldIx != FIELD_NONE; fieldIx = input.readFieldNumber(this)) {
            mergeFrom(input, message, fieldIx);
        }
    }

    public void mergeFrom(Input input, compiler.Pet message, int fieldIx) throws IOException {
        switch (fieldIx) {
            case FIELD_NONE:
                return;
            case FIELD_ID:
                message.setId(input.readInt32());
                break;
            case FIELD_NAME:
                message.setName(input.readString());
                break;
            case FIELD_DAMAGE:
                message.setDamage(input.readInt32());
                break;
            case FIELD_HP:
                message.setHp(input.readInt32());
                break;
            default:
                input.handleUnknownField(fieldIx, this);
        }
    }


    private static int[] FIELDS_TO_WRITE = { FIELD_ID, FIELD_NAME, FIELD_DAMAGE, FIELD_HP };

    public int[] getWriteFields() { return FIELDS_TO_WRITE; }

    public void writeTo(Output output, compiler.Pet message) throws IOException {
        int[] toWrite = getWriteFields();
        for (int i = 0; i < toWrite.length; i++) {
            writeTo(output, message, toWrite[i]);
        }
    }

    public void writeTo(Output output, compiler.Pet message, int fieldIx) throws IOException {
        switch (fieldIx) {
            case FIELD_NONE:
                break;
            case FIELD_ID:
                output.writeInt32(FIELD_ID, message.getId(), false);
                break;
            case FIELD_NAME:
                output.writeString(FIELD_NAME, message.getName(), false);
                break;
            case FIELD_DAMAGE:
                output.writeInt32(FIELD_DAMAGE, message.getDamage(), false);
                break;
            case FIELD_HP:
                output.writeInt32(FIELD_HP, message.getHp(), false);
                break;
            default:
                break;
        }
    }

    public String getFieldName(int number) {
        switch(number) {
            case FIELD_ID: return "id";
            case FIELD_NAME: return "name";
            case FIELD_DAMAGE: return "damage";
            case FIELD_HP: return "hp";
            default: return null;
        }
    }

    public int getFieldNumber(String name) {
        final Integer number = fieldMap.get(name);
        return number == null ? 0 : number.intValue();
    }

    final java.util.Map<String, Integer> fieldMap = new java.util.HashMap<String, Integer>(); {
        fieldMap.put("id", FIELD_ID);
        fieldMap.put("name", FIELD_NAME);
        fieldMap.put("damage", FIELD_DAMAGE);
        fieldMap.put("hp", FIELD_HP);
    }
}

```

For HeroModel you will get HeroModelSchema.java:

```
// Generated by http://code.google.com/p/protostuff/ ... DO NOT EDIT!
// Generated from proto

package compiler.schema;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import com.dyuproject.protostuff.Input;
import com.dyuproject.protostuff.Output;
import com.dyuproject.protostuff.Schema;
import com.dyuproject.protostuff.UninitializedMessageException;
import compiler.*;

public class HeroModelSchema
       implements Schema<HeroModel> {


    static final HeroModel DEFAULT_INSTANCE = new HeroModel();
    static final Schema<HeroModel> SCHEMA = new HeroModelSchema();

    public static HeroModel getDefaultInstance() { return DEFAULT_INSTANCE; }
    public static Schema<HeroModel> getSchema() { return SCHEMA; }

    public static final int FIELD_NONE = 0;
    public static final int FIELD_ID = 1;
    public static final int FIELD_NAME = 2;
    public static final int FIELD_HP = 3;
    public static final int FIELD_MP = 4;
    public static final int FIELD_PETS = 5;
    public static final int FIELD_WEAPONS = 6;
    public static final int FIELD_POTIONS = 7;

    public HeroModelSchema() {}


    public HeroModel newMessage() {
        return new HeroModel();
    }

    public Class<HeroModel> typeClass() {
        return HeroModel.class;
    }

    public String messageName() {
        return HeroModel.class.getSimpleName();
    }

    public String messageFullName() {
        return HeroModel.class.getName();
    }


    public boolean isInitialized(HeroModel message) {
        return true;
    }

    public boolean isPet(Item petsEntry) {
        return false;
    }
    public boolean isWeapon(Item weaponsEntry) {
        return false;
    }
    public boolean isPotion(Item potionsEntry) {
        return false;
    }

    public void mergeFrom(Input input, HeroModel message) throws IOException {
        for (int fieldIx = input.readFieldNumber(this); fieldIx != FIELD_NONE; fieldIx = input.readFieldNumber(this)) {
            mergeFrom(input, message, fieldIx);
        }
    }

    public void mergeFrom(Input input, HeroModel message, int fieldIx) throws IOException {
        switch (fieldIx) {
            case FIELD_NONE:
                return;
            case FIELD_ID:
                message.setId(input.readInt32());
                break;
            case FIELD_NAME:
                message.setName(input.readString());
                break;
            case FIELD_HP:
                message.setHp((short)input.readInt32());
                break;
            case FIELD_MP:
                message.setMp((short)input.readInt32());
                break;
            case FIELD_PETS:
                if (message.getItems() == null)
                    message.setItems(new ArrayList<Item>());
                message.getItems().add(input.mergeObject(null, PetSchema.getSchema()));
                break;

            case FIELD_WEAPONS:
                if (message.getItems() == null)
                    message.setItems(new ArrayList<Item>());
                message.getItems().add(input.mergeObject(null, WeaponSchema.getSchema()));
                break;

            case FIELD_POTIONS:
                if (message.getItems() == null)
                    message.setItems(new ArrayList<Item>());
                message.getItems().add(input.mergeObject(null, PotionSchema.getSchema()));
                break;

            default:
                input.handleUnknownField(fieldIx, this);
        }
    }


    private static int[] FIELDS_TO_WRITE = { FIELD_ID, FIELD_NAME, FIELD_HP, FIELD_MP, FIELD_PETS, FIELD_WEAPONS, FIELD_POTIONS };

    public int[] getWriteFields() { return FIELDS_TO_WRITE; }

    public void writeTo(Output output, HeroModel message) throws IOException {
        int[] toWrite = getWriteFields();
        for (int i = 0; i < toWrite.length; i++) {
            writeTo(output, message, toWrite[i]);
        }
    }

    public void writeTo(Output output, HeroModel message, int fieldIx) throws IOException {
        switch (fieldIx) {
            case FIELD_NONE:
                break;
            case FIELD_ID:
                output.writeInt32(FIELD_ID, message.getId(), false);
                break;
            case FIELD_NAME:
                output.writeString(FIELD_NAME, message.getName(), false);
                break;
            case FIELD_HP:
                output.writeInt32(FIELD_HP, (short)message.getHp(), false);
                break;
            case FIELD_MP:
                output.writeInt32(FIELD_MP, (short)message.getMp(), false);
                break;
            case FIELD_PETS:
                if (message.getItems() != null) {
                    for (Item petsEntry : message.getItems()) {
                        if (petsEntry != null && isPet(petsEntry))
                            output.writeObject(FIELD_PETS, (compiler.Pet)petsEntry, PetSchema.getSchema(), true);
                    }
                }

                break;
            case FIELD_WEAPONS:
                if (message.getItems() != null) {
                    for (Item weaponsEntry : message.getItems()) {
                        if (weaponsEntry != null && isWeapon(weaponsEntry))
                            output.writeObject(FIELD_WEAPONS, (compiler.Weapon)weaponsEntry, WeaponSchema.getSchema(), true);
                    }
                }

                break;
            case FIELD_POTIONS:
                if (message.getItems() != null) {
                    for (Item potionsEntry : message.getItems()) {
                        if (potionsEntry != null && isPotion(potionsEntry))
                            output.writeObject(FIELD_POTIONS, (compiler.Potion)potionsEntry, PotionSchema.getSchema(), true);
                    }
                }

                break;
            default:
                break;
        }
    }

    public String getFieldName(int number) {
        switch(number) {
            case FIELD_ID: return "id";
            case FIELD_NAME: return "name";
            case FIELD_HP: return "hitpoints";
            case FIELD_MP: return "manapoints";
            case FIELD_PETS: return "pets";
            case FIELD_WEAPONS: return "weapons";
            case FIELD_POTIONS: return "potions";
            default: return null;
        }
    }

    public int getFieldNumber(String name) {
        final Integer number = fieldMap.get(name);
        return number == null ? 0 : number.intValue();
    }

    final java.util.Map<String, Integer> fieldMap = new java.util.HashMap<String, Integer>(); {
        fieldMap.put("id", FIELD_ID);
        fieldMap.put("name", FIELD_NAME);
        fieldMap.put("hitpoints", FIELD_HP);
        fieldMap.put("manapoints", FIELD_MP);
        fieldMap.put("pets", FIELD_PETS);
        fieldMap.put("weapons", FIELD_WEAPONS);
        fieldMap.put("potions", FIELD_POTIONS);
    }
}

```

And for HeroModelSchema you will need to create a subclass to override check function in repeated items fields:

```
package compiler.schema;

import com.dyuproject.protostuff.Schema;
import compiler.*;

public class HeroSchema extends HeroModelSchema {
    static final compiler.Hero DEFAULT_INSTANCE = new compiler.Hero();
    static final Schema<HeroModel> SCHEMA = new HeroSchema();

    public static compiler.Hero getDefaultInstance() { return DEFAULT_INSTANCE; }
    public static Schema<compiler.HeroModel> getSchema() { return SCHEMA; }


    @Override
    public HeroModel newMessage() {
        return new Hero();
    }

    @Override
    public boolean isPet(Item item) {
        // this can be replace with item delegate item.isPet()
        // or item.isTypeOf(TYPE_PET);
        return item instanceof Pet;
    }

    @Override
    public boolean isWeapon(Item item) {
        return item instanceof Weapon;
    }

    @Override
    public boolean isPotion(Item item) {
        return item instanceof Potion;
    }
}

```