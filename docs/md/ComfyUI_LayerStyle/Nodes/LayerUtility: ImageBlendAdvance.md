---
tags:
- Image
- ImageBlend
- ImageComposite
---

# LayerUtility: ImageBlendAdvance
## Documentation
- Class name: `LayerUtility: ImageBlendAdvance`
- Category: `😺dzNodes/LayerUtility`
- Output node: `False`

The ImageBlendAdvance node is designed for advanced blending of images within a layer-based editing context. It leverages sophisticated algorithms to merge images seamlessly, offering users enhanced control over the blending process for creating complex visual compositions.
## Input types
### Required
- **`background_image`**
    - The base image over which the layer image will be blended. It serves as the canvas for the blending operation.
    - Comfy dtype: `IMAGE`
    - Python dtype: `Image`
- **`layer_image`**
    - The image to be blended onto the background image. This layer can be manipulated through various parameters to achieve the desired blending effect.
    - Comfy dtype: `IMAGE`
    - Python dtype: `Image`
- **`invert_mask`**
    - A boolean flag that, when set, inverts the blending mask, affecting how the layer image merges with the background.
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `bool`
- **`blend_mode`**
    - Defines the algorithm used for blending the layer image with the background, influencing the visual outcome of the blend.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `BlendMode`
- **`opacity`**
    - Determines the transparency level of the layer image, allowing for finer control over its visibility against the background.
    - Comfy dtype: `INT`
    - Python dtype: `int`
- **`x_percent`**
    - Specifies the horizontal positioning of the layer image relative to the background, enabling precise alignment.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`y_percent`**
    - Specifies the vertical positioning of the layer image relative to the background, enabling precise alignment.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`mirror`**
    - Allows for the mirroring of the layer image either horizontally or vertically, adding to the creative possibilities.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `MirrorMode`
- **`scale`**
    - Controls the size of the layer image, enabling scaling up or down as needed for the composition.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`aspect_ratio`**
    - Adjusts the aspect ratio of the layer image, maintaining its proportions while scaling.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`rotate`**
    - Rotates the layer image around its center, offering additional compositional flexibility.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`transform_method`**
    - Specifies the interpolation method used during transformations such as scaling or rotating, affecting image quality.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `TransformMethod`
- **`anti_aliasing`**
    - Sets the level of anti-aliasing applied to the layer image, enhancing the visual quality of edges and transitions.
    - Comfy dtype: `INT`
    - Python dtype: `int`
### Optional
- **`layer_mask`**
    - unknown
    - Comfy dtype: `MASK`
    - Python dtype: `unknown`
## Output types
- **`image`**
    - Comfy dtype: `IMAGE`
    - The final blended image resulting from the application of the specified parameters and blending operations.
    - Python dtype: `Image`
- **`mask`**
    - Comfy dtype: `MASK`
    - An optional output that provides the mask used in the blending process, useful for further editing or analysis.
    - Python dtype: `Mask`
## Usage tips
- Infra type: `GPU`
- Common nodes: unknown


## Source code
```python
class ImageBlendAdvance:

    def __init__(self):
        pass

    @classmethod
    def INPUT_TYPES(self):

        mirror_mode = ['None', 'horizontal', 'vertical']
        method_mode = ['lanczos', 'bicubic', 'hamming', 'bilinear', 'box', 'nearest']
        return {
            "required": {
                "background_image": ("IMAGE", ),  #
                "layer_image": ("IMAGE",),  #
                "invert_mask": ("BOOLEAN", {"default": True}),  # 反转mask
                "blend_mode": (chop_mode,),  # 混合模式
                "opacity": ("INT", {"default": 100, "min": 0, "max": 100, "step": 1}),  # 透明度
                "x_percent": ("FLOAT", {"default": 50, "min": -999, "max": 999, "step": 0.01}),
                "y_percent": ("FLOAT", {"default": 50, "min": -999, "max": 999, "step": 0.01}),
                "mirror": (mirror_mode,),  # 镜像翻转
                "scale": ("FLOAT", {"default": 1, "min": 0.01, "max": 100, "step": 0.01}),
                "aspect_ratio": ("FLOAT", {"default": 1, "min": 0.01, "max": 100, "step": 0.01}),
                "rotate": ("FLOAT", {"default": 0, "min": -999999, "max": 999999, "step": 0.01}),
                "transform_method": (method_mode,),
                "anti_aliasing": ("INT", {"default": 0, "min": 0, "max": 16, "step": 1}),
            },
            "optional": {
                "layer_mask": ("MASK",),  #
            }
        }

    RETURN_TYPES = ("IMAGE", "MASK")
    RETURN_NAMES = ("image", "mask")
    FUNCTION = 'image_blend_advance'
    CATEGORY = '😺dzNodes/LayerUtility'

    def image_blend_advance(self, background_image, layer_image,
                            invert_mask, blend_mode, opacity,
                            x_percent, y_percent,
                            mirror, scale, aspect_ratio, rotate,
                            transform_method, anti_aliasing,
                            layer_mask=None
                            ):
        b_images = []
        l_images = []
        l_masks = []
        ret_images = []
        ret_masks = []
        for b in background_image:
            b_images.append(torch.unsqueeze(b, 0))
        for l in layer_image:
            l_images.append(torch.unsqueeze(l, 0))
            m = tensor2pil(l)
            if m.mode == 'RGBA':
                l_masks.append(m.split()[-1])
            else:
                l_masks.append(Image.new('L', m.size, 'white'))
        if layer_mask is not None:
            if layer_mask.dim() == 2:
                layer_mask = torch.unsqueeze(layer_mask, 0)
            l_masks = []
            for m in layer_mask:
                if invert_mask:
                    m = 1 - m
                l_masks.append(tensor2pil(torch.unsqueeze(m, 0)).convert('L'))

        max_batch = max(len(b_images), len(l_images), len(l_masks))
        for i in range(max_batch):
            background_image = b_images[i] if i < len(b_images) else b_images[-1]
            layer_image = l_images[i] if i < len(l_images) else l_images[-1]
            _mask = l_masks[i] if i < len(l_masks) else l_masks[-1]
            # preprocess
            _canvas = tensor2pil(background_image).convert('RGB')
            _layer = tensor2pil(layer_image)

            if _mask.size != _layer.size:
                _mask = Image.new('L', _layer.size, 'white')
                log(f"Warning: {NODE_NAME} mask mismatch, dropped!", message_type='warning')

            orig_layer_width = _layer.width
            orig_layer_height = _layer.height
            _mask = _mask.convert("RGB")

            target_layer_width = int(orig_layer_width * scale)
            target_layer_height = int(orig_layer_height * scale * aspect_ratio)

            # mirror
            if mirror == 'horizontal':
                _layer = _layer.transpose(Image.FLIP_LEFT_RIGHT)
                _mask = _mask.transpose(Image.FLIP_LEFT_RIGHT)
            elif mirror == 'vertical':
                _layer = _layer.transpose(Image.FLIP_TOP_BOTTOM)
                _mask = _mask.transpose(Image.FLIP_TOP_BOTTOM)

            # scale
            _layer = _layer.resize((target_layer_width, target_layer_height))
            _mask = _mask.resize((target_layer_width, target_layer_height))
            # rotate
            _layer, _mask, _ = image_rotate_extend_with_alpha(_layer, rotate, _mask, transform_method, anti_aliasing)

            # 处理位置
            x = int(_canvas.width * x_percent / 100 - _layer.width / 2)
            y = int(_canvas.height * y_percent / 100 - _layer.height / 2)

            # composit layer
            _comp = copy.copy(_canvas)
            _compmask = Image.new("RGB", _comp.size, color='black')
            _comp.paste(_layer, (x, y))
            _compmask.paste(_mask, (x, y))
            _compmask = _compmask.convert('L')
            _comp = chop_image(_canvas, _comp, blend_mode, opacity)

            # composition background
            _canvas.paste(_comp, mask=_compmask)

            ret_images.append(pil2tensor(_canvas))
            ret_masks.append(image2mask(_compmask))

        log(f"{NODE_NAME} Processed {len(ret_images)} image(s).", message_type='finish')
        return (torch.cat(ret_images, dim=0), torch.cat(ret_masks, dim=0),)

```
