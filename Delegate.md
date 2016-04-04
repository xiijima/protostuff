**`com.dyuproject.protostuff.runtime.Delegate` is available since 1.0.7**(protostuff-runtime)

Useful for:
  1. singletons
  1. pojos with no fields
  1. pojos that should not be merged
  1. pojos that prefer to be written as scalar/inline values
  1. could be used to pack primitive array types into a byte array
  1. overriding protostuff-runtime's internal serializers for scalar fields (delegates have the first priority).

You basically register them on initialization via:
  * `DefaultIdStrategy.registerDelegate(Delegate<V> delegate)`
  * `ExplicitIdStrategy.Registry.registerDelegate(Delegate<V> delegate, int id)`
  * `IncrementalIdStrategy.Registry.registerDelegate(Delegate<V> delegate, int id)`

Here's an example for 1 or 2.
```

    public static final class Singleton
    {
        public static final Singleton INSTANCE = new Singleton();
        
        private Singleton() {}
        
        @Override
        public boolean equals(Object obj)
        {
            return this == obj && obj == INSTANCE;
        }
        
        public int hashCode()
        {
            return System.identityHashCode(this);
        }
    }
    
    public static final Delegate<Singleton> SINGLETON_DELEGATE = 
            new Delegate<Singleton>()
    {
        public Class<?> typeClass()
        {
            return Singleton.class;
        }
        
        public FieldType getFieldType()
        {
            return FieldType.UINT32;
        }
        
        public void writeTo(Output output, int number, Singleton value, 
                boolean repeated) throws IOException
        {
            output.writeUInt32(number, 0, repeated);
        }
        
        public Singleton readFrom(Input input) throws IOException
        {
            if(0 != input.readUInt32())
                throw new ProtostuffException("Corrupt input.");
            
            return Singleton.INSTANCE;
        }
        
        public void transfer(Pipe pipe, Input input, Output output, int number, boolean repeated)
                throws IOException
        {
            output.writeUInt32(number, input.readUInt32(), repeated);
        }
    };

```

Here's an example for 3 or 4.
```
    public static final Delegate<UUID> UUID_DELEGATE = new Delegate<UUID>()
    {

        public FieldType getFieldType()
        {
            return FieldType.BYTES;
        }

        public Class<?> typeClass()
        {
            return UUID.class;
        }
        
        public UUID readFrom(Input input) throws IOException
        {
            final byte[] buf = input.readByteArray();
            if(buf.length != 16)
                throw new ProtostuffException("Corrupt input.");
            
            return new UUID(toInt64(buf, 0), toInt64(buf, 8));
        }

        public void writeTo(Output output, int number, UUID value, boolean repeated)
                throws IOException
        {
            final byte[] buf = new byte[16];
            
            writeInt64(value.getMostSignificantBits(), buf, 0);
            writeInt64(value.getLeastSignificantBits(), buf, 8);
            
            output.writeByteArray(number, buf, repeated);
        }

        public void transfer(Pipe pipe, Input input, Output output, int number, 
                boolean repeated) throws IOException
        {
            input.transferByteRangeTo(output, false, number, repeated);
        }
    };
    
    static long toInt64(final byte[] buffer, int offset)
    {
        final byte b1 = buffer[offset++];
        final byte b2 = buffer[offset++];
        final byte b3 = buffer[offset++];
        final byte b4 = buffer[offset++];
        final byte b5 = buffer[offset++];
        final byte b6 = buffer[offset++];
        final byte b7 = buffer[offset++];
        final byte b8 = buffer[offset];
        
        return (((long)b8 & 0xff)    ) |
             (((long)b7 & 0xff) <<  8) |
             (((long)b6 & 0xff) << 16) |
             (((long)b5 & 0xff) << 24) |
             (((long)b4 & 0xff) << 32) |
             (((long)b3 & 0xff) << 40) |
             (((long)b2 & 0xff) << 48) |
             (((long)b1 & 0xff) << 56);
    }
    
    static void writeInt64(final long value, final byte[] buffer, int offset)
    {
        buffer[offset++] = (byte)(value >>> 56);
        buffer[offset++] = (byte)(value >>> 48);
        buffer[offset++] = (byte)(value >>> 40);
        buffer[offset++] = (byte)(value >>> 32);
        buffer[offset++] = (byte)(value >>> 24);
        buffer[offset++] = (byte)(value >>> 16);
        buffer[offset++] = (byte)(value >>>  8);
        buffer[offset] = (byte)(value >>>  0);
    }
```

Here's an example for 5.
```
    public static final Delegate<short[]> SHORT_ARRAY_DELEGATE = new Delegate<short[]>()
    {
        public Class<?> typeClass()
        {
            return short[].class;
        }
        
        public FieldType getFieldType()
        {
            return FieldType.BYTES;
        }
        
        public void writeTo(Output output, int number, short[] value, 
                boolean repeated) throws IOException
        {
            byte[] buffer = new byte[value.length*2];
            for(int i = 0, offset = 0; i < value.length; i++)
            {
                short s = value[i];
                buffer[offset++] = (byte)((s >>>  8) & 0xFF);
                buffer[offset++] = (byte)((s >>>  0) & 0xFF);
            }
            
            output.writeByteArray(number, buffer, repeated);
        }
        
        public short[] readFrom(Input input) throws IOException
        {
            byte[] buffer = input.readByteArray();
            short[] s = new short[buffer.length/2];
            for(int i = 0, offset = 0; i < s.length; i++)
            {
                s[i] = (short)((buffer[offset++] & 0xFF) << 8 | (buffer[offset++] & 0xFF));
            }
            
            return s;
        }
        
        public void transfer(Pipe pipe, Input input, Output output, int number, 
                boolean repeated) throws IOException
        {
            input.transferByteRangeTo(output, false, number, repeated);
        }
    };
```