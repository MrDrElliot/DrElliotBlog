+++
title = "Network Serialization in Unreal Engine C++"
date = 2024-08-16T10:28:54-05:00
+++

Hello, I wanted to talk about some ways you can improve networking performance in C++ by understanding and implementing efficient network serialization in Unreal Engine. Effective serialization is critical for transmitting data across the network without unnecessary overhead, especially in multiplayer games where bandwidth is limited.

This article assumes you're using Unreal Engine 5.2 or newer.

### What is Network Serialization?

Serialization in the context of networking refers to the process of converting data into a format that can be easily transmitted over a network and reconstructed later. In Unreal Engine, this process is crucial for ensuring that game states, variables, and objects are accurately synchronized between clients and servers.

### Unreal Engine's Built-in Serialization

Unreal Engine provides built-in support for serialization with its powerful reflection system. This system can automatically serialize properties marked with specific macros, making it easy to sync variables across the network.

Here’s an example of a simple struct that can be serialized:

```cpp
USTRUCT(BlueprintType)
struct FPlayerData
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Player Data")
    int32 Health;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Player Data")
    int32 Armor;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Player Data")
    FVector Location;

    // Serializes the data for network transmission
    friend FArchive& operator<<(FArchive& Ar, FPlayerData& Data)
    {
        Ar << Data.Health;
        Ar << Data.Armor;
        Ar << Data.Location;

        return Ar;
    }
};
```

In this example, the `FPlayerData` struct contains the player's health, armor, and location. The `operator<<` overload is implemented to serialize these properties, ensuring that they can be packed into a network packet and sent over the network.

A common point of confusion when first learning about any type of serialization in Unreal is the use of the `<<` operator. This operator works both during *reading*, and *writing*. `FArchive` has functions to determine which mode it's in. Such as 
the `write` mode with `Ar.IsSaving` or the `read` mode with `Ar.IsLoading`. This enables you to control how things get serialized during both read and write operations.

Something important to node about overriding the `<<` operator, is that it's used for **ALL** types of serialization, not just networking. Don't worry, there are ways to specifically control serialization **ONLY** during network serialization.

Unreal has a powerful `Struct Trait` System, now forewarning, the syntax may look a bit weird. But here is an example of how we can declare a `UStruct` with a `Network Serializer`

```cpp

template<>
struct TStructOpsTypeTraits<FPlayerData> : public TStructOpsTypeTraitsBase2<FPlayerData>
{
	enum
	{
		WithNetSerializer = true,
	};
};

```

This `TStructOpsTypeTraits` lets you define very specific ways a structure gets used, looking inside of `Class.h` we can see all of the struct traits to cover custom aspects of `UStructs`

```cpp

/** type traits to cover the custom aspects of a script struct **/
template <class CPPSTRUCT>
struct TStructOpsTypeTraitsBase2
{
	enum
	{
		WithZeroConstructor            = false,                         // struct can be constructed as a valid object by filling its memory footprint with zeroes.
		WithNoInitConstructor          = false,                         // struct has a constructor which takes an EForceInit parameter which will force the constructor to perform initialization, where the default constructor performs 'uninitialization'.
		WithNoDestructor               = false,                         // struct will not have its destructor called when it is destroyed.
		WithCopy                       = !TIsPODType<CPPSTRUCT>::Value, // struct can be copied via its copy assignment operator.
		WithIdenticalViaEquality       = false,                         // struct can be compared via its operator==.  This should be mutually exclusive with WithIdentical.
		WithIdentical                  = false,                         // struct can be compared via an Identical(const T* Other, uint32 PortFlags) function.  This should be mutually exclusive with WithIdenticalViaEquality.
		WithExportTextItem             = false,                         // struct has an ExportTextItem function used to serialize its state into a string.
		WithImportTextItem             = false,                         // struct has an ImportTextItem function used to deserialize a string into an object of that class.
		WithAddStructReferencedObjects = false,                         // struct has an AddStructReferencedObjects function which allows it to add references to the garbage collector.
		WithSerializer                 = false,                         // struct has a Serialize function for serializing its state to an FArchive.
		WithStructuredSerializer       = false,                         // struct has a Serialize function for serializing its state to an FStructuredArchive.
		WithPostSerialize              = false,                         // struct has a PostSerialize function which is called after it is serialized
		WithNetSerializer              = false,                         // struct has a NetSerialize function for serializing its state to an FArchive used for network replication.
		WithNetDeltaSerializer         = false,                         // struct has a NetDeltaSerialize function for serializing differences in state from a previous NetSerialize operation.
		WithSerializeFromMismatchedTag = false,                         // struct has a SerializeFromMismatchedTag function for converting from other property tags.
		WithStructuredSerializeFromMismatchedTag = false,               // struct has an FStructuredArchive-based SerializeFromMismatchedTag function for converting from other property tags.
		WithPostScriptConstruct        = false,                         // struct has a PostScriptConstruct function which is called after it is constructed in blueprints
		WithNetSharedSerialization     = false,                         // struct has a NetSerialize function that does not require the package map to serialize its state.
		WithGetPreloadDependencies     = false,                         // struct has a GetPreloadDependencies function to return all objects that will be Preload()ed when the struct is serialized at load time.
		WithPureVirtual                = false,                         // struct has PURE_VIRTUAL functions and cannot be constructed when CHECK_PUREVIRTUALS is true
		WithFindInnerPropertyInstance  = false,							// struct has a FindInnerPropertyInstance function that can provide an FProperty and data pointer when given a property FName
		WithCanEditChange			   = false,							// struct has an editor-only CanEditChange function that can conditionally make child properties read-only in the details panel (same idea as UObject::CanEditChange)
	};

	static constexpr EPropertyObjectReferenceType WithSerializerObjectReferences = EPropertyObjectReferenceType::Conservative; // struct's Serialize method(s) may serialize object references of these types - default Conservative means unknown and object reference collector archives should serialize this struct 
};

```

To see a very good example of this system in use, we can look at `FVector_NetQuantize`, located inside of `NetSerialization.h`.

```cpp

USTRUCT(meta = (HasNativeMake = "/Script/Engine.KismetMathLibrary.MakeVector_NetQuantize", HasNativeBreak = "/Script/Engine.KismetMathLibrary.BreakVector_NetQuantize"))
struct FVector_NetQuantize : public FVector
{
	GENERATED_USTRUCT_BODY()

	FORCEINLINE FVector_NetQuantize()
	{}

	explicit FORCEINLINE FVector_NetQuantize(EForceInit E)
	: FVector(E)
	{}

	FORCEINLINE FVector_NetQuantize(double InX, double InY, double InZ)
	: FVector(InX, InY, InZ)
	{}

	FORCEINLINE FVector_NetQuantize(const FVector &InVec)
	{
		FVector::operator=(InVec);
	}

	bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
	{
		bOutSuccess = SerializePackedVector<1, 20>(*this, Ar);
		return true;
	}
};

template<>
struct TStructOpsTypeTraits< FVector_NetQuantize > : public TStructOpsTypeTraitsBase2< FVector_NetQuantize >
{
	enum 
	{
		WithNetSerializer = true,
		WithNetSharedSerialization = true,
	};
};

```

Brushing over the boilerplate code, we can see that `FVector_NetQuantize` implements the trait `WithNetSerializer = true`, and then the function:

```cpp
	bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
```

-is implemented, this gives us full access to the FArchive responsible for the network serialization.

Another important trait is `WithNetDeltaSerializer`, which enables you direct access to determine what changes between two `NetSerialize` function calls, `FFastArraySerializer` takes advantage of this.



### Implementing Network Serialization in a Player Class

Let’s take it a step further and see how this serialization works within a player class.

```cpp
void AMyPlayerState::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // Use DOREPLIFETIME to ensure these properties are replicated
    DOREPLIFETIME(AMyPlayerState, PlayerData);
}

void AMyPlayerState::Serialize(FArchive& Ar)
{
    Super::Serialize(Ar);

    Ar << PlayerData;
}
```

In this code snippet:

- `GetLifetimeReplicatedProps` is used to define the properties that should be replicated to clients. Here, `PlayerData` is marked for replication using `DOREPLIFETIME`.
- The `Serialize` function is overridden to handle custom serialization of the `PlayerData` struct.

### Optimizing Serialization for Performance

While Unreal Engine handles a lot of serialization automatically, there are ways to optimize it for better performance:

1. **Minimize Data Transmission**: Only serialize and send the data that is absolutely necessary. Avoid transmitting redundant or static data that doesn't change often.

2. **Use Compression**: If you’re sending large amounts of data, consider compressing it before serialization. Unreal Engine provides built-in functions like `FArchive` for this purpose.

3. **Custom Serialization**: For complex objects, implement custom serialization logic to control exactly how data is packed and unpacked, which can reduce the size of your network packets.

### Example: Custom Serialization with Compression

Here’s a quick example of custom serialization with compression:

```cpp
void AMyPlayerState::SerializeCompressedData(FArchive& Ar)
{
    if (Ar.IsSaving())
    {
        TArray<uint8> UncompressedData;
        FMemoryWriter MemoryWriter(UncompressedData, true);
        MemoryWriter << PlayerData;

        // Compress the data
        TArray<uint8> CompressedData;
        FCompression::CompressMemory(NAME_Zlib, CompressedData.GetData(), CompressedData.Max(), UncompressedData.GetData(), UncompressedData.Num());

        // Serialize compressed data
        Ar << CompressedData;
    }
    else if (Ar.IsLoading())
    {
        TArray<uint8> CompressedData;
        Ar << CompressedData;

        TArray<uint8> UncompressedData;
        FCompression::UncompressMemory(NAME_Zlib, UncompressedData.GetData(), UncompressedData.Max(), CompressedData.GetData(), CompressedData.Num());

        FMemoryReader MemoryReader(UncompressedData, true);
        MemoryReader << PlayerData;
    }
}
```

In this example, data is compressed using the Zlib algorithm before being serialized, reducing the amount of data transmitted over the network. When received, the data is uncompressed and deserialized back into the original format.


Let's say you have a very very large structure, but that structure might not always be full of every piece of information that can be packaged into it, do we still have to replicate all the properties in the struct even if they're empty?
**NO**.

We can take advantage of the NetSerialize and access to the FArchive responsible by implementing something called `Bit Counting` or ` Rep Bit Counting`

`Bit Counting` is when you pack informatin into a bit `uint8` `uint32` (or whatever you suits your requirements) and determine what needs to get packaged up during the `Ar.IsSaving` phase. Let's take a look at `FGameplayEffectContext` from that `Gameplay Ability System`

```cpp

bool FGameplayEffectContext::NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
{
	uint8 RepBits = 0;
	if (Ar.IsSaving())
	{
		if (bReplicateInstigator && Instigator.IsValid())
		{
			RepBits |= 1 << 0;
		}
		if (bReplicateEffectCauser && EffectCauser.IsValid() )
		{
			RepBits |= 1 << 1;
		}
		if (AbilityCDO.IsValid())
		{
			RepBits |= 1 << 2;
		}
		if (bReplicateSourceObject && SourceObject.IsValid())
		{
			RepBits |= 1 << 3;
		}
		if (Actors.Num() > 0)
		{
			RepBits |= 1 << 4;
		}
		if (HitResult.IsValid())
		{
			RepBits |= 1 << 5;
		}
		if (bHasWorldOrigin)
		{
			RepBits |= 1 << 6;
		}
	}

	Ar.SerializeBits(&RepBits, 7);

	if (RepBits & (1 << 0))
	{
		Ar << Instigator;
	}
	if (RepBits & (1 << 1))
	{
		Ar << EffectCauser;
	}
	if (RepBits & (1 << 2))
	{
		Ar << AbilityCDO;
	}
	if (RepBits & (1 << 3))
	{
		Ar << SourceObject;
	}
	if (RepBits & (1 << 4))
	{
		SafeNetSerializeTArray_Default<31>(Ar, Actors);
	}
	if (RepBits & (1 << 5))
	{
		if (Ar.IsLoading())
		{
			if (!HitResult.IsValid())
			{
				HitResult = TSharedPtr<FHitResult>(new FHitResult());
			}
		}
		HitResult->NetSerialize(Ar, Map, bOutSuccess);
	}
	if (RepBits & (1 << 6))
	{
		Ar << WorldOrigin;
		bHasWorldOrigin = true;
	}
	else
	{
		bHasWorldOrigin = false;
	}

	if (Ar.IsLoading())
	{
		AddInstigator(Instigator.Get(), EffectCauser.Get()); // Just to initialize InstigatorAbilitySystemComponent
	}	
	
	bOutSuccess = true;
	return true;
}

```


What in the world is this giant thing? Well it's actually pretty simple, you see `FGameplayEffectContext` is a very large struct. `size: 128, alignment: 8`, and we might not always have every bit of information packed into every one of those variables. So we can take advantage of `bit counting` to determine what actually needs to get serialized up. 



```cpp

	uint8 RepBits = 0;
	if (Ar.IsSaving())
	{
		if (bReplicateInstigator && Instigator.IsValid())
		{
			RepBits |= 1 << 0;
		}
		if (bReplicateEffectCauser && EffectCauser.IsValid() )
		{
			RepBits |= 1 << 1;
		}
		if (AbilityCDO.IsValid())
		{
			RepBits |= 1 << 2;
		}
		if (bReplicateSourceObject && SourceObject.IsValid())
		{
			RepBits |= 1 << 3;
		}
		if (Actors.Num() > 0)
		{
			RepBits |= 1 << 4;
		}
		if (HitResult.IsValid())
		{
			RepBits |= 1 << 5;
		}
		if (bHasWorldOrigin)
		{
			RepBits |= 1 << 6;
		}
	}
    
```

This part here, only happens when the `FArchive` is in the *write* mode, at that time, you determine which properties have relevant values, and simply mark a `flag` into the `uint8 RepBits`, stating that something exists at that point we may want later.

The next part happens for both *writing* and *reading*. At whichtime the `RepBits` is serialized to the number of entries we put into it, (in this case 7 bits), and then as we come through the struct, we look and see which properties are relevant, and if they are. We serialize them with the `<<` operator, or by calling the properties custom `NetSerialize` function. When this gets *read* on the other side, the order is maintained through the `RepBits`, so the reader has the
ability to unpack the structure in the correct order. Which is **Extremely* important. Both *reading* and *writing* needs to happen in the same order, due to how buffers work. We use the `uint8` to maintain a small way to send the data through the network which contains our information about what was actually read to the archiver.

```cpp

Ar.SerializeBits(&RepBits, 7);

	if (RepBits & (1 << 0))
	{
		Ar << Instigator;
	}
	if (RepBits & (1 << 1))
	{
		Ar << EffectCauser;
	}
	if (RepBits & (1 << 2))
	{
		Ar << AbilityCDO;
	}
	if (RepBits & (1 << 3))
	{
		Ar << SourceObject;
	}
	if (RepBits & (1 << 4))
	{
		SafeNetSerializeTArray_Default<31>(Ar, Actors);
	}
	if (RepBits & (1 << 5))
	{
		if (Ar.IsLoading())
		{
			if (!HitResult.IsValid())
			{
				HitResult = TSharedPtr<FHitResult>(new FHitResult());
			}
		}
		HitResult->NetSerialize(Ar, Map, bOutSuccess);
	}
	if (RepBits & (1 << 6))
	{
		Ar << WorldOrigin;
		bHasWorldOrigin = true;
	}
	else
	{
		bHasWorldOrigin = false;
	}

	if (Ar.IsLoading())
	{
		AddInstigator(Instigator.Get(), EffectCauser.Get()); // Just to initialize InstigatorAbilitySystemComponent
	}	
	
	bOutSuccess = true;
	return true;
    
```

### Conclusion

Efficient network serialization is essential for any multiplayer game developed in Unreal Engine. By understanding and optimizing how data is serialized, you can significantly reduce network overhead, leading to smoother and more responsive gameplay.

If you’re looking to enhance your game’s networking performance, start by analyzing what data you’re sending across the network and consider implementing custom serialization strategies to optimize it.

---

By following these guidelines and examples, you'll be well on your way to improving your Unreal Engine project's networking performance. Happy coding!