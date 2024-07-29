# LayerStyle: Stroke
## Documentation
- Class name: `LayerStyle: Stroke`
- Category: `😺dzNodes/LayerStyle`
- Output node: `False`

The `LayerStyle: Stroke` node is designed to apply a stroke effect to a layer image, allowing for customization of the stroke's appearance through various parameters such as color, width, and opacity. It operates by blending the layer image with a background image, optionally using a mask for more precise control over the effect's application.
## Input types
### Required
- **`background_image`**
    - The background image over which the layer image will be placed. It serves as the canvas for the stroke effect.
    - Comfy dtype: `IMAGE`
    - Python dtype: `IMAGE`
- **`layer_image`**
    - The layer image to which the stroke effect will be applied. This image is blended with the background based on the specified parameters.
    - Comfy dtype: `IMAGE`
    - Python dtype: `IMAGE`
- **`invert_mask`**
    - A boolean parameter that determines whether the mask applied to the layer image should be inverted, affecting the areas where the stroke effect is applied.
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `BOOLEAN`
- **`blend_mode`**
    - Defines the blending mode used to combine the layer image with the background, influencing the visual outcome of the stroke effect.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
- **`opacity`**
    - The opacity level of the stroke effect, allowing for adjustments in transparency from fully opaque to fully transparent.
    - Comfy dtype: `INT`
    - Python dtype: `INT`
- **`stroke_grow`**
    - The amount by which the stroke effect grows or shrinks the layer image, enabling the adjustment of the effect's spread.
    - Comfy dtype: `INT`
    - Python dtype: `INT`
- **`stroke_width`**
    - Specifies the width of the stroke effect, determining how thick or thin the stroke appears around the layer image.
    - Comfy dtype: `INT`
    - Python dtype: `INT`
- **`blur`**
    - The level of blur applied to the stroke effect, which can soften the edges of the stroke for a more subtle appearance.
    - Comfy dtype: `INT`
    - Python dtype: `INT`
- **`stroke_color`**
    - The color of the stroke effect, allowing for customization of the stroke's appearance to match the desired aesthetic.
    - Comfy dtype: `STRING`
    - Python dtype: `STRING`
### Optional
- **`layer_mask`**
    - An optional mask that can be applied to the layer image, providing more control over where the stroke effect is applied.
    - Comfy dtype: `MASK`
    - Python dtype: `MASK`
## Output types
- **`image`**
    - Comfy dtype: `IMAGE`
    - The resulting image after applying the stroke effect, combining the layer and background images with the specified stroke parameters.
    - Python dtype: `IMAGE`
## Usage tips
- Infra type: `CPU`
- Common nodes: unknown


## Source code
```python
class Stroke:

    def __init__(self):
        pass

    @classmethod
    def INPUT_TYPES(self):

        return {
            "required": {
                "background_image": ("IMAGE", ),  #
                "layer_image": ("IMAGE",),  #
                "invert_mask": ("BOOLEAN", {"default": True}),  # 反转mask
                "blend_mode": (chop_mode,),  # 混合模式
                "opacity": ("INT", {"default": 100, "min": 0, "max": 100, "step": 1}),  # 透明度
                "stroke_grow": ("INT", {"default": 0, "min": -999, "max": 999, "step": 1}),  # 收缩值
                "stroke_width": ("INT", {"default": 8, "min": 0, "max": 999, "step": 1}),  # 扩张值
                "blur": ("INT", {"default": 0, "min": 0, "max": 100, "step": 1}),  # 模糊
                "stroke_color": ("STRING", {"default": "#FF0000"}),  # 描边颜色
            },
            "optional": {
                "layer_mask": ("MASK",),  #
            }
        }

    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("image",)
    FUNCTION = 'stroke'
    CATEGORY = '😺dzNodes/LayerStyle'

    def stroke(self, background_image, layer_image,
                  invert_mask, blend_mode, opacity,
                  stroke_grow, stroke_width, blur, stroke_color,
                  layer_mask=None
                  ):

        b_images = []
        l_images = []
        l_masks = []
        ret_images = []
        for b in background_image:
            b_images.append(torch.unsqueeze(b, 0))
        for l in layer_image:
            l_images.append(torch.unsqueeze(l, 0))
            m = tensor2pil(l)
            if m.mode == 'RGBA':
                l_masks.append(m.split()[-1])
        if layer_mask is not None:
            if layer_mask.dim() == 2:
                layer_mask = torch.unsqueeze(layer_mask, 0)
            l_masks = []
            for m in layer_mask:
                if invert_mask:
                    m = 1 - m
                l_masks.append(tensor2pil(torch.unsqueeze(m, 0)).convert('L'))
        if len(l_masks) == 0:
            log(f"Error: {NODE_NAME} skipped, because the available mask is not found.", message_type='error')
            return (background_image,)

        max_batch = max(len(b_images), len(l_images), len(l_masks))

        grow_offset = int(stroke_width / 2)
        inner_stroke = stroke_grow - grow_offset
        outer_stroke = inner_stroke + stroke_width
        for i in range(max_batch):
            background_image = b_images[i] if i < len(b_images) else b_images[-1]
            layer_image = l_images[i] if i < len(l_images) else l_images[-1]
            _mask = l_masks[i] if i < len(l_masks) else l_masks[-1]

            # preprocess
            _canvas = tensor2pil(background_image).convert('RGB')
            _layer = tensor2pil(layer_image).convert('RGB')

            if _mask.size != _layer.size:
                _mask = Image.new('L', _layer.size, 'white')
                log(f"Warning: {NODE_NAME} mask mismatch, dropped!", message_type='warning')

            inner_mask = expand_mask(image2mask(_mask), inner_stroke, blur)
            outer_mask = expand_mask(image2mask(_mask), outer_stroke, blur)
            stroke_mask = subtract_mask(outer_mask, inner_mask)
            color_image = Image.new('RGB', size=_layer.size, color=stroke_color)
            blend_image = chop_image(_layer, color_image, blend_mode, opacity)
            _canvas.paste(_layer, mask=_mask)
            _canvas.paste(blend_image, mask=tensor2pil(stroke_mask))

            ret_images.append(pil2tensor(_canvas))

        log(f"{NODE_NAME} Processed {len(ret_images)} image(s).", message_type='finish')
        return (torch.cat(ret_images, dim=0),)

```
