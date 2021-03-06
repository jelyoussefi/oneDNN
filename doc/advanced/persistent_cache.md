Persistent Cache{#dev_guide_persistent_cache}
===========================================================

oneDNN provides [primitive cache](@ref dev_guide_primitive_cache) to reduce
primitive creation overhead. However, the primitive cache has no effect when
a primitive is created for the first time. For some applications it can be
critical to reduce that overhead.

oneDNN provides an API that can be used to create a persistent cache. User can
use that API to obtain a cache blob ID from a primitive descriptor and a
cache blob from a primitive to use them as a key and value respectively. The
cache blob contains data that can be used to speed up primitive creation.

@note
Content and size of the cache blob ID and cache blob objects are not specified.

* The cache blob ID can be obtained via @ref dnnl::primitive_desc_base::get_cache_blob_id
* The cache blob can be obtained via @ref dnnl::primitive::get_cache_blob
* Each primitive class provides a constructor that takes the cache blob along
with the primitive descriptor.

@note
oneDNN version and git commit hash (@ref dnnl_version_t::hash) affect equality
of the cache blob IDs. That is, the cache blob ID obtained from a primitive
descriptor will be different for different oneDNN versions and git commit
hashes.

@warning
The git commit hash may not be available if the git package was not found during
a CMake call. In this case, the cache blob ID will be the same for different
hashes. This may result in fetching a wrong cache blob from persistent cache.

## Relation to Primitive Cache
In the case when a primitive is created from a cache blob and the identical
primitive is present in the primitive cache the one from primitive cache will
be returned to the user, and the given cache blob will not be used. Otherwise,
the cache blob will be used to speed up the primitive creation. The information
about how the primitive was created (`cache_miss`, `cache_hit` or
`from_cache_blob`) is part of the verbose output for verbose level 2
(@ref dev_guide_verbose).

## API Usage Example

The following pseudo-code demonstrates a simple example of persistent cache
implementation using the oneDNN API:

~~~cpp
using namespace dnnl;

{
    convolution_forward::primitive_desc conv_pd(desc, attr, engine);
    convolution_forward conv(conv_pd);

    std::vector<uint8_t> key = conv_pd.get_cache_blob_id();
    std::vector<uint8_t> value = conv.get_cache_blob();
    store_cache_blob_on_disk(key, value);
}

{
    convolution_forward::primitive_desc conv_pd(desc, attr, engine);
    std::vector<uint8_t> key = conv_pd.get_cache_blob_id();
    std::vector<uint8_t> value = load_cache_blob_from_disk(key);
    convolution_forward conv_from_cache_blob(conv_pd, value);
}
~~~

## Limitations

The API is implemented for the OpenCL runtime only. For CPU engine kind and
other runtimes the library will return #dnnl_unimplemented in the case of the C
API or throw a corresponding @ref dnnl::error exception in the case of the C++
API.
