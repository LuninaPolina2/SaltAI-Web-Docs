---
tags:
- Prompt
---

# IF Chat Prompt👨‍💻
## Documentation
- Class name: `IF_ChatPrompt`
- Category: `ImpactFrames💥🎞️`
- Output node: `True`

The IF_ChatPrompt node is designed to facilitate interactive chat experiences by generating responses based on user inputs and a variety of parameters. It supports customization of the chat experience through different engines, models, and prompts, and includes features for managing conversation history and context.
## Input types
### Required
- **`prompt`**
    - The initial prompt or question to start the conversation. It sets the context for the chatbot's response.
    - Comfy dtype: `STRING`
    - Python dtype: `str`
- **`base_ip`**
    - The IP address of the server where the chat engine is hosted. It's crucial for establishing a connection to the engine.
    - Comfy dtype: `STRING`
    - Python dtype: `str`
- **`port`**
    - The port number on the server to connect to the chat engine. It specifies the exact entry point for communication.
    - Comfy dtype: `STRING`
    - Python dtype: `str`
- **`engine`**
    - Specifies the chat engine to use for generating responses. It allows for selection among various pre-defined engines, enabling flexibility in response generation.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `list[str]`
- **`selected_model`**
    - The specific model selected for generating responses. This parameter allows for further customization of the chat experience.
    - Comfy dtype: `[]`
    - Python dtype: `tuple`
- **`assistant`**
    - Defines the assistant's persona or role in the conversation, allowing for tailored responses based on the selected assistant.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `list[str]`
### Optional
- **`context`**
    - The current context or state of the conversation. It's essential for generating coherent and contextually relevant responses.
    - Comfy dtype: `STRING`
    - Python dtype: `str`
- **`images`**
    - An image to be included in the chat context, allowing for responses that can incorporate visual elements.
    - Comfy dtype: `IMAGE`
    - Python dtype: `Image`
- **`max_tokens`**
    - The maximum number of tokens to generate for the response. It controls the length of the chatbot's replies.
    - Comfy dtype: `INT`
    - Python dtype: `int`
- **`temperature`**
    - Controls the randomness of the response generation, affecting the creativity and variability of the replies.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`top_k`**
    - Limits the number of highest probability vocabulary tokens considered for each step, influencing the response's diversity.
    - Comfy dtype: `INT`
    - Python dtype: `int`
- **`top_p`**
    - Nucleus sampling parameter that controls the cumulative probability cutoff, shaping the response's unpredictability.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`repeat_penalty`**
    - Adjusts the likelihood of repeating the same line, aiming to reduce redundancy in responses.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
- **`stop`**
    - A set of tokens that signal the end of a response, helping to delineate the chatbot's replies.
    - Comfy dtype: `STRING`
    - Python dtype: `str`
- **`seed`**
    - A seed for the random number generator, ensuring reproducibility of responses under the same conditions.
    - Comfy dtype: `INT`
    - Python dtype: `int`
- **`random`**
    - Toggles between using a fixed seed or temperature for randomness, affecting the consistency of responses.
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `bool`
- **`embellish_prompt`**
    - A prompt modifier that adds embellishment to the initial prompt, enhancing the creativity of the response.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `list[str]`
- **`style_prompt`**
    - A prompt modifier that applies a specific style to the response, allowing for stylistic customization.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `list[str]`
- **`neg_prompt`**
    - A prompt modifier that specifies content to avoid in the response, helping to tailor the chatbot's output.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `list[str]`
- **`clear_history`**
    - Controls whether to clear the chat history, affecting the continuity of the conversation.
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `bool`
- **`history_steps`**
    - Specifies the number of recent messages to retain in the chat history, influencing the context available for generating responses.
    - Comfy dtype: `INT`
    - Python dtype: `int`
- **`keep_alive`**
    - Determines whether to keep the model loaded between requests, impacting response time and resource usage.
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `bool`
- **`text_cleanup`**
    - unknown
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `unknown`
- **`mode`**
    - unknown
    - Comfy dtype: `BOOLEAN`
    - Python dtype: `unknown`
## Output types
- **`Question`**
    - Comfy dtype: `STRING`
    - The original question or prompt provided to the chatbot.
    - Python dtype: `str`
- **`Response`**
    - Comfy dtype: `STRING`
    - The generated response from the chatbot based on the input prompt and parameters.
    - Python dtype: `str`
- **`Negative`**
    - Comfy dtype: `STRING`
    - A generated response that specifically avoids the content outlined in the neg_prompt parameter.
    - Python dtype: `str`
- **`Context`**
    - Comfy dtype: `STRING`
    - The updated context of the conversation after the response, including any modifications made during processing.
    - Python dtype: `str`
## Usage tips
- Infra type: `CPU`
- Common nodes: unknown


## Source code
```python
class IFChatPrompt:
    
    RETURN_TYPES = ("STRING", "STRING", "STRING", "STRING",)
    RETURN_NAMES = ("Question", "Response", "Negative", "Context",)
    FUNCTION = "describe_picture"
    OUTPUT_NODE = True
    CATEGORY = "ImpactFrames💥🎞️"

    @classmethod
    def INPUT_TYPES(cls):
        node = cls()
        return {
            "required": {
                "prompt": ("STRING", {"multiline": True, "default": ""}),
                "base_ip": ("STRING", {"default": node.base_ip}),
                "port": ("STRING", {"default": node.port}),
                "engine": (["ollama", "kobold", "lms", "textgen", "groq", "openai", "anthropic"], {"default": node.engine}),
                #"selected_model": (node.get_models("node.engine", node.base_ip, node.port), {}), 
                "selected_model": ((), {}),
                "assistant": ([name for name in node.assistants.keys()], {"default": node.assistant}),
            },
            "optional": {
                "context": ("STRING", {"forceInput": True}),
                "images": ("IMAGE", ),
                "max_tokens": ("INT", {"default": 2048, "min": 1, "max": 8192}),
                "temperature": ("FLOAT", {"default": 0.7, "min": 0.0, "max": 2.0, "step": 0.1}),
                "top_k": ("INT", {"default": 40, "min": 0, "max": 100}),
                "top_p": ("FLOAT", {"default": 0.2, "min": 0.0, "max": 1.0, "step": 0.1}),
                "repeat_penalty": ("FLOAT", {"default": 1.1, "min": 0.0, "max": 10.0, "step": 0.1}),
                "stop": ("STRING", {"default": "<|end_of_text|>", "multiline": False}),
                "seed": ("INT", {"default": 94687328150, "min": 0, "max": 0xffffffffffffffff}),
                "random": ("BOOLEAN", {"default": False, "label_on": "Seed", "label_off": "Temperature"}),
                "embellish_prompt": ([name for name in node.embellish_prompts.keys()], {}),
                "style_prompt": ([name for name in node.style_prompts.keys()], {}),
                "neg_prompt": ([name for name in node.neg_prompts.keys()], {}),
                "clear_history": ("BOOLEAN", {"default": True, "label_on": "Clear History", "label_off": "Keep History"}),
                "history_steps": ("INT", {"default": 10, "min": 0, "max": 0xffffffffffffffff}),
                "keep_alive": ("BOOLEAN", {"default": False, "label_on": "Keeps_Model", "label_off": "Unloads_Model"}),
                "text_cleanup": ("BOOLEAN", {"default": True, "label_on": "Apply", "label_off": "Raw Text"}),
                "mode": ("BOOLEAN", {"default": True, "label_on": "Mode: SD", "label_off": "Mode: Chat"}),
                #"external_api_key": ("STRING", {"default": "", "multiline": False}),
            },
            "hidden": {
                "model": ("STRING", {"default": ""}),
            },
        }

    @classmethod
    def IS_CHANGED(cls, engine, base_ip, port, assistant, keep_alive, seed, random, history_steps, selected_model):
        node = cls()
        seed_changed = seed != node.seed or random != node.random
        engine_changed = engine != node.engine
        base_ip_changed = base_ip != node.base_ip
        port_changed = port != node.port
        selected_model_changed = node.selected_model != node.get_models(engine, base_ip, port)

        if seed_changed or engine_changed or base_ip_changed or port_changed or selected_model_changed:
            node.engine = engine
            node.base_ip = base_ip
            node.port = port
            node.selected_model = node.get_models(engine, base_ip, port)
            node.assistant = assistant
            node.keep_alive = keep_alive
            node.seed = seed
            node.random = random
            node.history_steps = history_steps
            return True
        """if engine == "textgen":
            if node.selected_model != selected_model:
                node.load_model_textgen(selected_model, base_ip, port)
            return True"""
        return False
    
    
    def __init__(self):
        self.base_ip = "localhost" 
        self.port = "11434"     
        self.engine = "ollama" 
        self.selected_model = ""
        self.context = None
        self.assistant = "Cortana"
        self.comfy_dir = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
        self.presets_dir = os.path.join(os.path.dirname(__file__), "presets")
        self.assistants_file = os.path.join(self.presets_dir, "assistants.json")
        self.assistants = self.load_presets(self.assistants_file)
        self.neg_prompts_file = os.path.join(self.presets_dir, "neg_prompts.json")
        self.embellish_prompts_file = os.path.join(self.presets_dir, "embellishments.json")
        self.style_prompts_file = os.path.join(self.presets_dir, "style_prompts.json")
        self.neg_prompts = self.load_presets(self.neg_prompts_file)
        self.embellish_prompts = self.load_presets(self.embellish_prompts_file)
        self.style_prompts = self.load_presets(self.style_prompts_file)
        self.keep_alive = False
        self.seed = 94687328150
        self.chat_history = []
        self.history_steps = 10
        #self.external_api_key = ""


    def load_presets(self, file_path):
        with open(file_path, 'r') as f:
            presets = json.load(f)
        return presets
   
    def get_api_key(self, api_key_name, engine):
        if engine not in ["ollama", "kobold", "lms", "textgen"]:  
            api_key = os.getenv(api_key_name)
            if api_key:
                return api_key
        else:
            print(f'you are using ollama as the engine, no api key is required')
    
    """def load_model_textgen(self, selected_model, base_ip, port):
        headers = {"Content-Type": "application/json"}
        api_url = f'http://{base_ip}:{port}/v1/internal/model/load'   
        data = {
            "model_name": selected_model,
            "args": {"load_in_4bit": True }
        }
        try:
            response = requests.post(api_url, headers=headers, json=data)
            response.raise_for_status()
            print(f"Model {selected_model} loaded successfully.")
        except Exception as e:
            print(f"Failed to load model {selected_model}: {e}")"""
    
    def get_models(self, engine, base_ip, port):
        if engine == "groq":   
            return ["gemma-7b-it", "llama2-70b-4096", "llama3-70b-8192", "llama3-8b-8192","mixtral-8x7b-32768"]
        
        elif engine == "ollama":
            api_url = f'http://{base_ip}:{port}/api/tags'
            try:
                response = requests.get(api_url)
                response.raise_for_status()
                models = [model['name'] for model in response.json().get('models', [])]
                return models
            except Exception as e:
                print(f"Failed to fetch models from Ollama: {e}")
                return []
        elif engine == "lms":
            api_url = f'http://{base_ip}:{port}/v1/models'
            try:
                response = requests.get(api_url)
                if response.status_code == 200:
                    data = response.json()
                    models = [model['id'] for model in data['data']]
                    return models
                else:
                    print(f"Failed to fetch models from LM Studio. Status code: {response.status_code}")
                    return []
            except requests.exceptions.RequestException as e:
                print(f"Error connecting to LM Studio server: {e}")
                return []
        elif engine == "textgen":
            api_url = f'http://{base_ip}:{port}/v1/internal/model/list'
            try:
                response = requests.get(api_url)
                response.raise_for_status()
                models = response.json()['model_names']
                return models
            except Exception as e:
                print(f"Failed to fetch models from text-generation-webui: {e}")
                return []
        elif engine == "kobold":
            api_url = f'http://{base_ip}:{port}/api/v1/model'
            try:
                response = requests.get(api_url)
                response.raise_for_status()
                model = response.json()['result']
                return [model]
            except Exception as e:
                print(f"Failed to fetch models from Kobold: {e}")
                return []
        elif engine == "anthropic":
            return ["claude-3-opus-20240229", "claude-3-sonnet-20240229", "claude-3-haiku-20240307"]
        elif engine == "openai":
            api_key = self.get_api_key("OPENAI_API_KEY", engine)
            headers = {
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json"
            }
            api_url = "https://api.openai.com/v1/models"
            try:
                response = requests.get(api_url, headers=headers)
                response.raise_for_status()
                models = [model["id"] for model in response.json()["data"]]
                return models
            except Exception as e:
                print(f"Failed to fetch models from OpenAI: {e}")
                return []
        else:
            print(f"Unsupported engine - {engine}")
            return []
    
    def prepare_messages(self, prompt, assistant, images=None):
        assistant_content = self.assistants.get(assistant, "")
        image_message = textwrap.dedent("""
                Analyze the images provided, search for relevant details from the images to include on your response.
                Reply to the user's specific question or prompt and include relevant details extracted from the images.
            """)
        if images is not None:
            system_message = f"{assistant_content}\n{image_message}"
            
        else:
            system_message = f"{assistant_content}"
   
        
        user_message = prompt if prompt.strip() != "" else "Please provide a general description of the images."

        messages = []

        # Add the conversation history and user message regardless of history being empty
        for message in self.chat_history:
            messages.append({"role": message["role"], "content": message["content"]})

        messages.append({"role": "user", "content": user_message})

        return user_message, system_message, messages


    def describe_picture(self, prompt, engine, selected_model, base_ip, port, assistant, neg_prompt, embellish_prompt, style_prompt, 
                         temperature=0.7, max_tokens=2048, seed=0, random=False, history_steps=10, keep_alive=False, top_k=40, top_p=0.2, 
                         repeat_penalty=1.1, stop="", context=None, images=None, mode=True, clear_history=True, text_cleanup=True):
        
        embellish_content = self.embellish_prompts.get(embellish_prompt, "")
        style_content = self.style_prompts.get(style_prompt, "")
        neg_content = self.neg_prompts.get(neg_prompt, "")

        if images is not None:
            # Normalize tensor 0-255
            img_np = 255.0 * images[0].cpu().numpy()
            # Clip the values to the valid range [0, 255]
            img = Image.fromarray(np.clip(img_np, 0, 255).astype(np.uint8))
            
            # Resize the image if it's too large
            max_size = (1024, 1024)  # Adjust the maximum size as needed
            img.thumbnail(max_size)
            
            # Create a BytesIO object to store the image data
            buffered = io.BytesIO()
            
            # Save the resized image as PNG
            img.save(buffered, format="PNG")
            
            base64_image = base64.b64encode(buffered.getvalue()).decode("utf-8")
        else:
            base64_image = None

        available_models = self.get_models(engine, base_ip, port)
        if available_models is None or selected_model not in available_models:
            error_message = f"Invalid model selected: {selected_model} for engine {engine}. Available models: {available_models}"
            print(error_message)
            raise ValueError(error_message)

        user_message, system_message, messages = self.prepare_messages(prompt, assistant, images)
        if clear_history:
            self.chat_history = []
            context = None
        else:
            self.chat_history = self.chat_history[-history_steps:] if history_steps > 0 else []

        if engine == "ollama":
            if stop == "":
                stop = None 
            else:
                stop = ["\n", f"{stop}"]
        elif engine == "lms":
            if stop == "":
                stop = None 
            else:
                stop = ["\n", f"{stop}"]
        elif engine == "kobold":
            if stop == "":
                stop = None 
            else:
                stop = ["\n\n\n\n\n", f"{stop}"]
        else:
            stop = None
        try:
            generated_text, context = self.send_request(engine, base_ip, port, base64_image, selected_model, system_message, user_message, messages, seed, 
                                                        temperature, max_tokens, random, top_k, top_p, repeat_penalty, stop, keep_alive, context)           
            
            if text_cleanup:
                generated_text = process_text(generated_text)
            else:
                generated_text = generated_text      

            description = f"{generated_text}".strip()
            if not clear_history:   
                context = context
                self.chat_history.append({"role": "user", "content": user_message})
                self.chat_history.append({"role": "assistant", "content": description})
            else:
                context = None
                self.chat_history = []        
            """print("Conversation History:")
            for message in self.chat_history:
                role = message["role"]
                content = message["content"]
                print(f"{role.capitalize()}: {content}")"""
            if mode == False:
                return prompt, description, neg_prompt, context
            else:
                combined_prompt = f"{embellish_content} {description} {style_content}"
                return prompt, combined_prompt, neg_content, context
        except Exception as e:
            print(f"Exception occurred: {e}")
            return "Exception occurred while processing images.", ""      

    def send_request(self, engine, base_ip, port, base64_image, selected_model, system_message, user_message, messages, seed, 
                     temperature, max_tokens, random, top_k, top_p, repeat_penalty, stop, keep_alive, context=None):
    
        api_functions = {
            "groq": send_groq_request,
            "anthropic": send_anthropic_request,
            "openai": send_openai_request,
            "kobold": send_kobold_request,
            "ollama": send_ollama_request,
            "lms": send_lmstudio_request,
            "textgen": send_textgen_request
        }

        if engine not in api_functions:
            raise ValueError(f"Invalid engine: {engine}")

        api_function = api_functions[engine]

        
        if engine == "ollama":
            response, context = api_function(f"http://{base_ip}:{port}/api/generate", base64_image, 
                                    selected_model, system_message, user_message, messages, seed, 
                                    temperature, max_tokens, random, top_k, top_p, repeat_penalty, stop, keep_alive, context)
        elif engine == "kobold":
            response = api_function(f"http://{base_ip}:{port}/v1/chat/completions", base64_image, selected_model, system_message,
                                     user_message, messages, seed, temperature, max_tokens, top_k, top_p, repeat_penalty, stop)
            context = None
        elif engine == "lms":
            response = api_function(f"http://{base_ip}:{port}/v1/chat/completions", base64_image, selected_model, system_message,
                                     user_message, messages, seed, temperature, max_tokens, top_k, top_p, repeat_penalty, stop)
            context = None
        elif engine == "textgen":
            response = api_function(f"http://{base_ip}:{port}/v1/chat/completions", base64_image, selected_model, system_message,
                                     user_message, messages, seed, temperature, max_tokens, top_k, top_p, repeat_penalty, stop)
            context = None
        elif engine == "openai":
            api_key = self.get_api_key(f"{engine.upper()}_API_KEY", engine)
            response = api_function(base64_image, selected_model, system_message, user_message, messages, api_key, 
                                    seed, temperature, max_tokens, top_p, repeat_penalty)
            context = None
        else:
            api_key = self.get_api_key(f"{engine.upper()}_API_KEY", engine)
            response = api_function(selected_model, system_message, user_message, messages, api_key, temperature, 
                                    max_tokens, base64_image)
            context = None

        # Update chat history after receiving the response
        self.chat_history.append({"role": "user", "content": user_message})
        self.chat_history.append({"role": "assistant", "content": response})

        return response, context

```
