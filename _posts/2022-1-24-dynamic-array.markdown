---
layout: post
title: Dynamic Arrays [Part 1]
description: Discussion about dynamic arrays and a possible implementation
img: darray.png # Add image post (optional)
tags: [Game Engine, Programming, Containers] # add tag
---

## Intro

Hello again! I'm taking a brief detour from my originally planned sequence because I felt like talking about containers today. The first one that's important for the engine is the dynamic array. In general, I try to use static arrays whenever possible, but for convenience and learning I wanted to add a dynamic array implementation. 

*Note: This is just one implementation, it is by no means the best way to do it, this is just what works for me.*

You might be wondering why I would bother writing a container from scratch when things like std::vector exist. There are two main reasons, one, I want to learn as much as possible so reinventing/reimplementing the wheel is the name of the game. Two, I want as few external dependencies as reasonably possible, and as much control over how my code interacts with the metal as I can. The dependencies I'm trying to avoid include the STL so we'll be rolling our own containers and the like.

## Partial Interface
*Types.h*
{% highlight c++ %}
// Unsigned int types
typedef unsigned char  u8;
typedef unsigned short u16;
typedef unsigned u32;
typedef unsigned long long u64;

// Signed int types
typedef char i8;
typedef short i16;
typedef int i32;
typedef long long i64;

// Floating point types
typedef float f32;
typedef double f64;

// Boolean types
typedef char b8;
typedef int b32;
{% endhighlight %}

*DArray.h*
{% highlight c++ %}
#define DARRAY_DEFAULT_CAPACITY 1

void* _Darray_Create( u64 stride, u64 initCapacity );
void _Darray_Destroy( void* darray );

void* _Darray_PushBack( void* darray, const void* value );
void* _Darray_Resize( void* darray );

#define Darray_Create(type) _Darray_Create(sizeof(type), DARRAY_DEFAULT_CAPACITY)

#define Darray_Reserve(type, capacity) _Darray_Create(sizeof(type), capacity)

#define Darray_Destroy(darray) _Darray_Destroy(darray)

#define Darray_PushBack(darray, value)                    \
  {                                                    \
    decltype(value) temp = value;                       \
    darray = (decltype(darray))_Darray_PushBack(darray, &temp); \
  }   
{% endhighlight %}

I'm going to talk through the most basic functionality that I need out of a dynamic array in this section. Later, as needs arise I will add further utilities and features.

*Note: I will probably ditch using decltype at some point, also I'm using the convention of putting an underscore before internal functions, the actual interface functions are the macros.*

The bare minimum I need to be able to do is create/destroy the dynamic array (with or without a starting capacity), push_back a value, and resize the buffer when it gets too small. 

## Implementation

*DArray.cpp*
{% highlight c++ %}
struct DarrayHeader
{
  u64 m_Capacity;
  u64 m_Length;
  u64 m_Stride;
};

struct DarrayStats
{
  u64 m_Allocations;
};

DarrayStats g_Stats;
{% endhighlight %}

### Memory Layout

![Mem Layout](/assets/img/darray_mem_layout.png) 

The additional overhead from a plain c-style array is minimal, there is an additional 24 bytes that are just before the internal buffer in memory. The pointer to the internal buffer is what is passed back to the user and what they interact with. There are three fields in the header, m_Stride is the size of each element in the array, m_Length is the number of elements currently in the array, and m_Capacity is the total number of **allocated** bytes / m_Stride.

### Create

*DArray.cpp*
{% highlight c++ %}
DarrayHeader* _Darray_GetHeader( void* darray )
{
  return (DarrayHeader*)((char*)darray - sizeof( DarrayHeader ));
}

void* _Darray_Create( u64 stride, u64 initCapacity )
{
  u64 dataSize = initCapacity * stride;
  u64 headerSize = sizeof( DarrayHeader );

  void* arr = Memory_Allocate( headerSize + dataSize, MEMORY_TAG_DYNARRAY );
  Memory_ZeroMemory( arr, headerSize + dataSize );
  DarrayHeader* header = (DarrayHeader*)arr;
  header->m_Capacity = initCapacity;
  header->m_Length = 0;
  header->m_Stride = stride;

#if USING(DARRAY_DEBUG_STATS)
  ++g_Stats.m_Allocations;
#endif

  return (char*)arr + headerSize;
}
{% endhighlight %}

We haven't covered the memory system yet, or the platform layer, but without going into too much depth, for windows Memory_Allocate and Memory_ZeroMemory are basically just wrappers for malloc and memset with 0's. 

Here we need to calculate how much memory we need to allocate. We have the actual buffer which is the size of each element (or stride) x the initial capacity of the array (1 if user doesn't call reserve with a specific amount). We then add the additional size of the header and allocate a chunk of that size. In the future there will be specific memory pre-allocated by a memory manager that will allow for this to not cause an OS level allocation which we want to avoid as much as possible. 

We then zero that memory to for good measure, fill out the header info and return the user the pointer to the actual data buffer. The only additional bit I have in here is the start of some debug tracking code that will get stripped out of distribution builds.  

### Destroy
*DArray.cpp*
{% highlight c++ %}
void _Darray_Destroy( void* darray )
{
  DarrayHeader* header = _Darray_GetHeader( darray );
  Memory_Free( header, header->m_Capacity * header->m_Stride + sizeof(DarrayHeader), MEMORY_TAG_DYNARRAY );
}
{% endhighlight %}

Nothing fancy here, calculating the size of the buffer to free and freeing it. I should probably save the sizeof(DarrayHeader) somewhere so I can stop recalculating it. Also, it may be worthwhile to add some debug validation checks to detect double free's or bad pointers at some point.

### Push Back

*DArray.cpp*
{% highlight c++ %}
void* _Darray_PushBack( void* darray, const void* value )
{
  DarrayHeader* header = _Darray_GetHeader( darray );
  // resize if at capacity
  if ( header->m_Length == header->m_Capacity )
  {
    darray = _Darray_Resize( darray );
    header = _Darray_GetHeader( darray );
  }

  // insert value
  u64 insertOffset = header->m_Length * header->m_Stride;
  char* insertPosition = (char*)darray + insertOffset;
  Memory_CopyMemory( insertPosition, value, header->m_Stride );
  ++header->m_Length;

  // return new or existing array
  return darray;
}
{% endhighlight %}

Fairly straight forward here as well, we do an initial check to see if we are at capacity, if we are, we will resize the buffer and update the pointer, otherwise, we calculate the position to insert at and memcpy the new value into it and update the length.

### Resize

*DArray.cpp*
{% highlight c++ %}
#define DARRAY_GROWTH_RATE 2

void* _Darray_Resize( void* darray )
{
DarrayHeader* header = _Darray_GetHeader( darray );
  u64 curLength = header->m_Length;
  u64 curBufferSize = header->m_Capacity * header->m_Stride + sizeof( DarrayHeader );

  void* newDarray = _Darray_Create( header->m_Stride, header->m_Capacity * DARRAY_GROWTH_RATE );
  Memory_CopyMemory( newDarray, header, curBufferSize );
  DarrayHeader* newHeader = _Darray_GetHeader( newDarray );
  newHeader->m_Capacity = header->m_Capacity * DARRAY_GROWTH_RATE;
  Memory_Free( header, curBufferSize, MEMORY_TAG_DYNARRAY );

  return newDarray;
}
{% endhighlight %}

Resize is called when we have reached the capacity of a buffer and need to expand it. We create a new array with capacity = the current capacity x the growth rate for arrays (2 at the moment). We memcpy the old data into the new array and update the capacity, then free the old memory, and return the new array.

## Use Case

{% highlight c++ %}
u64* buffer;
buffer = (u64*)Darray_Create( u64 );
for ( u32 i = 0; i < 100; ++i )
{
    Darray_PushBack( buffer, i );
}
{% endhighlight %}

Here's a quick example using the functionality we got to today. We create a buffer of u64's then PushBack the numbers 0-99, causing 7 resizes to occur. The first real system that we're going to use this for is the event system, when we get to that it will become easier to see why this is useful. Until then, I will write up another post with the rest of the functionality I need right now for dynamic arrays, then we will resume the regular order of posts, picking back up with Logging!