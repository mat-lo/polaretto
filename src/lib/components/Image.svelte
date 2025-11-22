<script lang="ts">
  import type { ImageData } from '../types';

  let {
    src, // ImageData object, pre-loaded by preprocessor
    alt,
    sizes,
    width,
    height,
    fit, // Add this
    loading = 'lazy',
    class: className = '',
    style = '',
    ...restProps
  }: { 
    src: ImageData;
    alt: string;
    sizes?: string;
    width?: number;
    height?: number;
    fit?: 'cover' | 'contain' | 'fill' | 'inside' | 'outside'; // Add type
    loading?: 'lazy' | 'eager';
    class?: string;
    style?: string;
    [key: string]: any;
  } = $props();

  // Use first source's srcset (typically AVIF or WebP)
  let srcset = $derived(src?.sources[0]?.srcset || '');
  let finalSizes = $derived(sizes || src?.sources[0]?.sizes || '100vw');

  let placeholderStyle = $derived.by(() => {
    if (!src?.placeholder) return '';

    if (src.placeholder.startsWith('data:image/')) {
      return `background-image: url(${src.placeholder}); background-size: cover; background-position: center;`;
    } else if (src.placeholder.startsWith('rgb(')) {
      return `background-color: ${src.placeholder};`;
    }
    return '';
  });

  // Merge placeholder style with user style and explicit height
  let finalStyle = $derived([
    placeholderStyle, 
    height ? `height: ${height}px` : '', 
    style
  ].filter(Boolean).join('; '));
</script>

<!-- No conditional - data is always available for perfect SSR -->
<img
  src={src.src}
  {alt}
  {srcset}
  sizes={finalSizes}
  width={width || src.width}
  height={height || src.height}
  {loading}
  class={className}
  style={finalStyle}
  {...restProps}
/>

<style>
  img {
    max-width: 100%;
    height: auto;
    display: block;
  }
</style>