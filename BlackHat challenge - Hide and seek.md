# Link for the challange 
https://dojo-yeswehack.com/challenge-of-the-month/blackhat-2025

## **Spoiler-Free Guide**

This challenge is a great opportunity to explore how Python handles code execution and file management under the hood.

Here are some key things to focus on:

- Understand how Python reads and processes code — especially the difference between source files (.py) and compiled files (.pyc).
- Carefully analyze the challenge code — pay attention to how the payload is handled and what functions it passes through.
- Investigate the input possibilities — what can you actually do with the payload? How flexible is it?

To deepen your understanding and help you approach the challenge more effectively, check out these resources:
- https://coderivers.org/blog/python-markdown/
- https://facelessuser.github.io/pymdown-extensions/extensions/snippets/
  
These will give you insight into how Markdown rendering works in Python and how the pymdownx.snippets extension can be used to include external files
— a key mechanic in this challenge.
## **Code**

```
# Python 3.12.10
import os, sys, markdown
from urllib.parse import unquote
from jinja2 import Environment, FileSystemLoader
template = Environment(
    autoescape=True,
    loader=FileSystemLoader('/tmp/templates'),
).get_template('index.html')
os.chdir('/tmp')

sys.path.insert(0, os.getcwd())
from lib import hide

import_v('pymdownx.snippets', '9.11', 'pymdown-extensions')

def main():
    # Remove the flag - lets see if you can find me now!
    flag = ""
    hide.selfDestruct()

    payload = unquote("")

    msg = markdown.markdown(
        payload,
        extensions=['pymdownx.snippets'],
        extension_configs={
            'pymdownx.snippets': {
                'encoding': 'latin1'
            },
            'base_path': './templates',
            'check_paths': True
        }
    )
    resp = "You found the flag! &#x1F6A9;" if hide.validate(msg) else hide.notfound()

    print( template.render(msg=msg, msg_resp=resp) )

main()
```
## **Payload**
```
import os
os.chdir('/tmp')
os.mkdir('lib')
os.mkdir('snippets')
os.mkdir('templates')

with open('snippets/flag.md', 'w') as f:
    f.write('Where can I found the flag?!')

with open('lib/hide.py', 'w') as f:
    f.write('''
msg = "Keep looking, there was nothing in the "
def selfDestruct():
    from os import remove
    remove("/tmp/lib/hide.py")    

def notfound():
  from random import choice
  return choice([
    "Nothing to be found here! &#x1F440;",
    "Well... I used to be here &#x1F3C3;&#x1F4A8;",
    "That hiding place is useless, I'm not hiding there &#x1F926;",
    "Give me five, you didn't found me! &#x1F91A;",
    "Don't walk in there! &#x1F62C;",
    "Didn't you already check that place? &#x1F928;",
    "Keep looking, I'm grabbing some tea meanwhile &#x1F375;",
  ])

def validate(value):
  return "{flag}" in value

def kitchen():
    return msg+"kitchen!"

def locker():
    return "You found me! - {flag}"

def basement():
    return msg+"basement!"

def garage():
    return msg+"garage!"
'''.format(flag=flag).strip())

with open('templates/index.html', 'w') as f:
    f.write('''
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
  </head>
  <body class="bg-white">
    <div class="flex items-center justify-center min-h-screen bg-gray-100 bg-cover no-repeat bg-[url('https://static.vecteezy.com/ti/gratis-foton/p1/33330469-se-av-de-remsa-i-las-vegas-nevada-de-las-vegas-remsa-ar-ett-ungefar-4-2-mil-6-8-km-stracka-av-las-vegas-boulevard-las-vegas-remsa-natt-ai-genererad-gratis-fotona.jpg')]">
      <!-- Phone container -->
      <div
        class="relative w-[290px] h-[540px] bg-black rounded-[40px] shadow-2xl border-[12px] border-black shadow-xl"
      >
        <!-- Camera dot -->
        <div
          class="absolute top-2 left-1/2 -translate-x-1/2 w-3 h-3 bg-gray-800 rounded-full z-20"
        ></div>

        <!-- Top dash (status bar) -->
        <div
          class="absolute top-1 left-0 w-full px-4 flex justify-between items-center text-[10px] text-gray-500 z-10"
        >
          <!-- Left icons -->
          <div class="flex items-center gap-1">
            <span>9:41</span>
          </div>
          <!-- Right icons -->
          <div class="flex items-center gap-1">
            <div class="flex items-center gap-1">
              <span></span>
            </div>
            <div class="flex items-center gap-1">
              <span>&#x1FAAB;13.37%</span>
            </div>
          </div>
        </div>

        <!-- White screen (chat UI) -->
        <div
          class="w-full h-full rounded-[28px] bg-white flex flex-col justify-between p-4 pt-8 pb-2"
        >
          <!-- Chat messages -->
          <div
            class="flex flex-col mt-8 space-y-6 overflow-y-auto max-h-[360px]"
          >
            <div
              class="bg-gray-200 text-xl text-black p-2 rounded-xl max-w-[75%] rounded-bl-none"
            >
              Try to find me, I got your flag! &#x1F92B;
            </div>
            {% if msg != "" %}
              <div
                class="text-xl right-0 bg-blue-500 text-white p-2 rounded-xl max-w-[75%] self-end rounded-br-none"
              >
                <span>{{ msg | safe }}</span>
              </div>

              {% if msg_resp != "" %}
                <div
                  class="bg-gray-200 text-xl text-black p-2 rounded-xl max-w-[75%] rounded-bl-none"
                >
                  <span>{{ msg_resp | safe }}</span>
                </div>
              {% endif %}
            {% endif %}
          </div>

          <!-- Input field -->

          <form class="flex mt-2 mb-4 items-center max-w-sm mx-auto">
            <button
              type="submit"
              class="cursor-pointer p-2 ms-2 text-sm font-medium text-zinc-500 bg-zinc-300 rounded-full"
            >
              <svg
                class="w-4 h-4"
                aria-hidden="true"
                xmlns="http://www.w3.org/2000/svg"
                width="24"
                height="24"
                fill="none"
                viewBox="0 0 24 24"
              >
                <path
                  stroke="currentColor"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M5 12h14m-7 7V5"
                />
              </svg>
            </button>
            <input
              type="text"
              id="msg"
              class="ml-2 h-8 p-2 bg-white border border-2 border-gray-300 text-gray-900 text-sm rounded-full"
              placeholder="Message..."
            />
          </form>

          <!-- Gesture line -->
          <div class="absolute bottom-2 left-1/2 -translate-x-1/2">
            <div class="w-20 h-1 bg-black rounded-full"></div>
          </div>
        </div>
      </div>
    </div>
    <img class="absolute w-auto h-26 p-6 bottom-0 left-0" src="https://www.blackhat.com/images/logo.png">
    <img class="absolute w-auto h-24 p-6 bottom-0 right-0" src="https://www.yeswehack.com/_next/static/media/YWHlogo.022c0d8e.svg">
  </body>
</html>
'''.strip())
```
## **Guide with Spoilers**

When you first look at the challenge, you’ll notice that the payload is URL-encoded. 
This encoding prevents you from easily modifying the Python code — special characters like quotes (") are transformed, making direct edits tricky.

One of the first things I noticed was the call to:
``` 
hide.selfDestruct()
```
This method removes the hide library, which is responsible for verifying the authenticity of the flag. So the question is:

    How can a method from a deleted library still be called?

Python is an interpreted language, meaning it reads and executes code at runtime. But when a module is imported, Python often compiles it into a .pyc file stored in a __pycache__ directory. 
This compiled version can still be used even if the original .py file is deleted.

Even though hide.py is deleted, its compiled version might still exist. And we know for sure that it’s still in memory because these calls work:
```
hide.validate(msg)
hide.notfound()
```
This means the module is loaded and its methods are accessible — we just need to find the compiled file.
Let’s look at the code that processes the payload:

```
    payload = unquote("")

    msg = markdown.markdown(
        payload,
        extensions=['pymdownx.snippets'],
        extension_configs={
            'pymdownx.snippets': {
                'encoding': 'latin1'
            },
            'base_path': './templates',
            'check_paths': True
        }
    )
    resp = "You found the flag! &#x1F6A9;" if hide.validate(msg) else hide.notfound()

    print( template.render(msg=msg, msg_resp=resp) )
```
This shows that the payload is rendered using the markdown() function with the pymdownx.snippets extension. This extension allows you to include external files using:
```
--8<-- "file"
```
- https://facelessuser.github.io/pymdown-extensions/extensions/snippets/
  
So naturally, I wondered: Can I read any file on the system?

I started with a known file path:

```
snippets/flag.md
``` 
![CTF BlackHat](https://github.com/GaTalim/dojoyeswehack-challenges-of-the-month/raw/main/images/ctf-blackHat-try.jpg)

But here's where it gets interesting.

I wanted to access a hidden library, but it had been deleted from the source code. So I thought: maybe the compiled version still exists somewhere?

In Python, compiled .pyc files are stored in a __pycache__ folder inside the parent directory of the original .py file. The naming convention is:

```
/__pycache__/Client.cpython-313.pyc
```
So if the original file was hide.py and the Python version is 3.12.10, the compiled file should be:
```
lib/__pycache__/hide.cpython-312.pyc
```
I tested this by trying to include it with the snippet syntax:
```
--8<-- "lib/__pycache__/hide.cpython-312.pyc"
```
![CTF BlackHat](https://github.com/GaTalim/dojoyeswehack-challenges-of-the-month/raw/main/images/ctf-blackhat-sucess.png)


 And boom — it worked! I got the output.


**FLAG{A_B1t_0f_F0r3nsic5_T0_F1nd_M3}**


**Remember:** Share knowledge, protect sensitive information, and help make the internet safer!

---

*Happy CTF hunting!*
