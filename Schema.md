# Schema #

Comprised of:
  * the serialization logic of an object
  * the deserialization logic of an object
  * the validation of an object's required fields
  * the mapping of an object's field names to field numbers
  * the instantiation of the object.

For existing objects, you can use [protostuff-runtime](ProtostuffRuntime.md) to generate the
schema for you to cache and use at runtime via reflections.

For the people who prefer to further customize the schema, like validation of some required fields, you can code it by hand.

Say for example, here is an exising domain object.

## `User.java` ##

```

public class User
{
    
    private String firstName;
    private String lastName;
    private String email;
    private List<User> friends;
    
    public User()
    {
    
    }
    
    public User(String email)
    {
        this.email = email;
    }
    
    // getters and setters
}

```

Here's what the hand-written schema would look like:

## `UserSchema.java` ##

```

public class UserSchema implements Schema<User>
{

    public boolean isInitialized(User user)
    {
        return user.getEmail() != null;
    }

    public void mergeFrom(Input input, User user) throws IOException
    {
        while(true)
        {
            int number = input.readFieldNumber(this);
            switch(number)
            {
                case 0:
                    return;
                case 1:
                    user.setEmail(input.readString());
                    break;
                case 2:
                    user.setFirstName(input.readString());
                    break;
                case 3:
                    user.setLastName(input.readString());
                    break;
                case 4:
                    if(message.friends == null)
                        message.friends = new ArrayList<User>();
                    message.friends.add(input.mergeObject(null, this));
                    break;
                default:
                    input.handleUnknownField(number, this);
            }
        }
    }

    public void writeTo(Output output, User user) throws IOException
    {
        if(user.getEmail() == null)
            throw new UninitializedMessageException(user, this);
        
        output.writeString(1, user.getEmail(), false);
        
        if(user.getFirstName() != null)
            output.writeString(2, user.getFirstName(), false);
        
        if(user.getLastName() != null)
            output.writeString(3, user.getLastName(), false);

        if(message.friends != null)
        {
            for(User friend : message.friends)
            {
                if(friend != null)
                    output.writeObject(4, friend, this, true);
            }
        }
    }

    public User newMessage()
    {
        return new User();
    }

    public Class<User> typeClass()
    {
        return User.class;
    }

    public String messageName()
    {
        return User.class.getSimpleName();
    }
    
    public String messageFullName()
    {
        return User.class.getName();
    }
    
    // the mapping between the field names to the field numbers.
    
    public String getFieldName(int number)
    {
        switch(number)
        {
            case 1:
                return "email";
            case 2:
                return "firstName";
            case 3:
                return "lastName";
            case 4:
                return "friends";
            default:
                return null;
        }
    }

    public int getFieldNumber(String name)
    {
        Integer number = fieldMap.get(name);
        return number == null ? 0 : number.intValue();
    }
    
    private static final HashMap<String,Integer> fieldMap = new HashMap<String,Integer>();    
    static
    {
        fieldMap.put("email", 1);
        fieldMap.put("firstName", 2);
        fieldMap.put("lastName", 3);
        fieldMap.put("friends", 4);
    }
}

```