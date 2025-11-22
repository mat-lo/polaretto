<script lang="ts">
  import type { ImageData } from '../types';

  let { 
    src, // This is ImageData, pre-loaded by preprocessor + Vite plugin
    alt,
    sizes,
    loading = 'lazy',
    class: className = '',
    style = '',
    width,
    height,
    fit,
    placeholder,
    artDirectives = [],
    ...restProps
  }: { 
    src: ImageData;
    alt: string;
    sizes?: string;
    width?: number;
    height?: number;
    fit?: 'cover' | 'contain' | 'fill' | 'inside' | 'outside';
    placeholder?: 'blur' | 'dominant-color' | 'traced-svg' | 'pixelated' | 'none';
    loading?: 'lazy' | 'eager';
    class?: string;
    style?: string;
    artDirectives?: Array<{ media: string; src: ImageData | string }>;
    [key: string]: any;
  } = $props();

  // Generate unique class name based on src path + props for SSR consistency
  const simpleHash = (str: string) => {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = (hash << 5) - hash + char;
      hash = hash & hash; 
    }
    return 'pic-' + (hash >>> 0).toString(16);
  };

  const uniqueClass = $derived(simpleHash(src.src + JSON.stringify(artDirectives) + alt));

  const getBgImage = (p: string | string[] | null | undefined) => {
    if (!p) return '';
    if (Array.isArray(p)) {
        return p.map(url => `url(${url})`).join(', ');
    }
    return `url(${p})`;
  };

  // Construct dynamic CSS for responsive placeholders
  let responsiveStyles = $derived.by(() => {
    let css = '';
    
    // Default (Main) Placeholder
    const mainBg = getBgImage(src.placeholder);
    if (mainBg) {
        css += `.${uniqueClass} { background-image: ${mainBg}; }`;
    }

    // Art Directives Placeholders
    if (artDirectives) {
        artDirectives.forEach(dir => {
           if (typeof dir.src !== 'string' && dir.src.placeholder) {
              const dirBg = getBgImage(dir.src.placeholder);
              css += `@media ${dir.media} { .${uniqueClass} { background-image: ${dirBg} !important; } }`;
           }
        });
    }
    // console.log('[Picture] Generated CSS:', css);
    return css;
  });

  // Base styles (layout + pixelated rendering)
  let baseStyle = $derived.by(() => {
    let s = `background-size: cover; background-position: center;`;
    
    if (height) {
        s += ` height: ${height}px;`;
    }

    if (placeholder === 'pixelated') {
      s += ' image-rendering: -moz-crisp-edges; image-rendering: pixelated;';
    }
    
    return [s, style].filter(Boolean).join('; ');
  });
</script>

<!-- Inject dynamic styles for responsive placeholders -->
{@html `<style>${responsiveStyles}</style>`}

<picture class={className}>
  <!-- Art direction sources (if provided) -->
  {#if artDirectives && artDirectives.length > 0}
    {#each artDirectives as directive}
      {#if typeof directive.src === 'string'}
        <source media={directive.media} srcset={directive.src} />
      {:else}
        {#each directive.src.sources as source}
          <source
            media={directive.media}
            type={source.type}
            srcset={source.srcset}
            sizes={sizes || source.sizes || '100vw'}
          />
        {/each}
      {/if}
    {/each}
  {/if}

  <!-- Format-specific sources -->
  {#each src.sources as source}
    <source
      type={source.type}
      srcset={source.srcset}
      sizes={sizes || source.sizes || '100vw'}
    />
  {/each}

  <!-- Fallback img -->
  <img
    src={src.src}
    {alt}
    width={src.width}
    height={src.height}
    {loading}
    class={uniqueClass}
    style={baseStyle}
    {...restProps}
  />
</picture>

<style>
  picture {
    display: block;
  }

  img {
    max-width: 100%;
    height: auto;
    display: block;
  }
</style>