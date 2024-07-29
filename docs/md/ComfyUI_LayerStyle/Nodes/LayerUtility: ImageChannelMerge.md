# LayerUtility: ImageChannelMerge
## Documentation
- Class name: `LayerUtility: ImageChannelMerge`
- Category: `😺dzNodes/LayerUtility`
- Output node: `False`

The ImageChannelMerge node is designed to merge separate image channels into a single image, supporting various color modes. It allows for the flexible combination of up to four image channels, accommodating optional fourth channel input for extended color space manipulation.
## Input types
### Required
- **`channel_i`**
    - Represents one of the image channels to be merged. Each channel contributes to the composite image by adding its unique layer of detail, color, or depth. The index 'i' can range from 1 to 4, where channels 1 through 3 are required and channel 4 is optional, allowing for more complex color space manipulations when included.
    - Comfy dtype: `IMAGE`
    - Python dtype: `torch.Tensor`
- **`mode`**
    - Specifies the color mode for the merged image, such as RGBA, YCbCr, LAB, or HSV. This determines how the channels are combined and interpreted, affecting the overall appearance of the final composite image.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
### Optional
## Output types
- **`image`**
    - Comfy dtype: `IMAGE`
    - The resulting image after merging the specified channels. It represents a composite image in the specified color mode.
    - Python dtype: `torch.Tensor`
## Usage tips
- Infra type: `GPU`
- Common nodes: unknown


## Source code
```python
class ImageChannelMerge:

    def __init__(self):
        pass

    @classmethod
    def INPUT_TYPES(self):
        channel_mode = ['RGBA', 'YCbCr', 'LAB', 'HSV']
        return {
            "required": {
                "channel_1": ("IMAGE", ),  #
                "channel_2": ("IMAGE",),  #
                "channel_3": ("IMAGE",),  #
                "mode": (channel_mode,),  # 通道设置
            },
            "optional": {
                "channel_4": ("IMAGE",),  #
            }
        }

    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("image",)
    FUNCTION = 'image_channel_merge'
    CATEGORY = '😺dzNodes/LayerUtility'

    def image_channel_merge(self, channel_1, channel_2, channel_3, mode, channel_4=None):

        c1_images = []
        c2_images = []
        c3_images = []
        c4_images = []
        ret_images = []

        width, height = tensor2pil(torch.unsqueeze(channel_1[0], 0)).size
        for c in channel_1:
            c1_images.append(torch.unsqueeze(c, 0))
        for c in channel_2:
            c2_images.append(torch.unsqueeze(c, 0))
        for c in channel_3:
            c3_images.append(torch.unsqueeze(c, 0))
        if channel_4 is not None:
            for c in channel_4:
                c4_images.append(torch.unsqueeze(c, 0))
        else:
            c4_images.append(Image.new('L', size=(width, height), color='white'))

        max_batch = max(len(c1_images), len(c2_images), len(c3_images), len(c4_images))
        for i in range(max_batch):
            c_1 = c1_images[i] if i < len(c1_images) else c1_images[-1]
            c_2 = c2_images[i] if i < len(c2_images) else c2_images[-1]
            c_3 = c3_images[i] if i < len(c3_images) else c3_images[-1]
            c_4 = c4_images[i] if i < len(c4_images) else c4_images[-1]
            ret_image = image_channel_merge((tensor2pil(c_1), tensor2pil(c_2), tensor2pil(c_3), tensor2pil(c_4)), mode)

            ret_images.append(pil2tensor(ret_image))

        log(f"{NODE_NAME} Processed {len(ret_images)} image(s).", message_type='finish')
        return (torch.cat(ret_images, dim=0),)

```
