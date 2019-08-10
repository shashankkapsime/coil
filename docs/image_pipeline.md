# Extending the Image Pipeline

Android supports many [image types](https://developer.android.com/guide/topics/media/media-formats) out of the box, however what if you need to add support for an unsupported format?

Fortunately, `ImageLoader`s support pluggable components to add new data types, new fetching behavior, new image encodings, or otherwise overwrite the base image loading behavior. Coil's image pipeline consists of three main parts: [Mappers](../api/coil-base/coil.map/-mapper), [Fetchers](../api/coil-base/coil.fetch/-fetcher), and [Decoders](../api/coil-base/coil.decode/-decoder).

Any custom components must be added to the `ImageLoader` when constructing it by its [ComponentRegistry](../api/coil-base/coil/-component-registry):

```kotlin
val imageLoader = ImageLoader(context) {
    componentRegistry {
        add(ItemMapper())
        add(ProtocolBufferFetcher())
        add(GifDecoder())
    }
}
```

## Mappers

Mappers allow you to add support for custom data types. For instance, say we get this model from our server:

```kotlin
data class Item(
    val id: Int,
    val imageUrl: String,
    val price: Int,
    val weight: Double
)
```

We could write a custom mapper to map it to an `HttpUrl`:

```kotlin
class ItemMapper : Mapper<Item, HttpUrl> {
    override fun map(data: Item): HttpUrl = HttpUrl.get(data.imageUrl)
}
```

After registering it when constructing our `ImageLoader` (see above), we can safely load an `Item`:

```kotlin
imageView.loadAny(item)
```

If you want to know a `Target`'s size when mapping an object, you can extend from [Measured Mapper](../api/coil-base/coil.map/-measured-mapper).

!!! Note
    Extending from `Measured Mapper` can prevent setting an image request's placeholder and setting cached `Drawable`s synchronously only for that mapper's data type. Prefer extending `Mapper` if you do not need the `Target`'s size.

See [Mapper](../api/coil-base/coil.map/-mapper) and [Measured Mapper](../api/coil-base/coil.map/-measured-mapper) for more information.

## Fetchers

Fetchers translate data into either a `BufferedSource` or a `Drawable`.

See [Fetcher](../api/coil-base/coil.fetch/-fetcher) for more information.

## Decoders

Decoders read a `BufferedSource` as input and return a `Drawable`. Use this interface to add support for custom file formats (e.g. GIF, SVG, TIFF, etc.).

See [Decoder](../api/coil-base/coil.decode/-decoder) for more information.

!!! Note
    Decoders are responsible for closing the `BufferedSource` when finished. This allows custom decoders to return a `Drawable` while still reading the source. This can be useful to support file types such as [progressive JPEG](https://www.liquidweb.com/kb/what-is-a-progressive-jpeg/) where there is incremental information to show.