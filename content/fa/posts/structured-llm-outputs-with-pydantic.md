+++
date = '2025-12-27T23:14:24+03:30'
draft = false
title = 'خروجی ساخت‌یافته از LLM با Pydantic و LangChain'
summary= "در این مطلب یاد می‌گیریم چطور با استفاده از Pydantic و LangChain خروجی‌های ساخت‌یافته و قابل‌اعتماد از LLMها بگیریم و آن‌ها را در کاربردهایی مثل NER و تولید مجموعه‌دادهٔ مصنوعی به‌کار ببریم."
+++

# مقدمه

در بسیاری از کاربردهای عملی LLMها، نیاز داریم خروجی ساخت‌یافته (مثل JSON، CSV و …) دریافت کنیم. مثال‌های رایج شامل ساخت مجموعه‌دادهٔ مصنوعی (Synthetic Dataset Generation)، استخراج اطلاعات، یا شناسایی موجودیت‌های نامدار (Named Entity Recognition یا NER) هستند.

یک راه ساده این است که در prompt، شِمای خروجی را برای مدل توضیح بدهیم؛ اما هیچ تضمینی وجود ندارد که مدل همیشه دقیقاً از این قالب پیروی کند. اینجاست که **Pydantic** به کمک ما می‌آید.

با استفاده از Pydantic می‌توانیم شِمای خروجی را به‌صورت صریح تعریف کنیم و خروجی LLM را اعتبارسنجی (validate) کنیم. اگر خروجی با شِما سازگار نباشد، یا خطا می‌گیریم و متوجه می‌شویم که ساختار رعایت نشده، یا (بسته به فریم‌ورک مورد استفاده) مدل دوباره فراخوانی می‌شود تا خروجی معتبر تولید کند.

---

# مثال: Named Entity Recognition

فرض کنید می‌خواهیم روی یک متن، NER انجام دهیم و موجودیت‌های `PERSON`، `ORG`، `LOC` و `DATE` را استخراج کنیم. ابتدا شِمای خروجی را با Pydantic تعریف می‌کنیم:

```python
from typing import List, Literal
from pydantic import BaseModel, Field

class Entity(BaseModel):
    text: str = Field(description="The exact text span of the entity")
    label: Literal["PERSON", "ORG", "LOC", "DATE"] = Field(
        description="The type of the named entity"
    )

class NEROutput(BaseModel):
    entities: List[Entity] = Field(
        description="List of named entities found in the text"
    )
```

سپس این شِما را به مدل زبانی می‌دهیم:

```python
from langchain.chat_models import init_chat_model

chat_model = init_chat_model(
    model=MODEL_NAME,
    temperature=0,
)

structured_chat_model = chat_model.with_structured_output(NEROutput)
```

در اینجا `temperature` را صفر گذاشته‌ایم، چون خروجی قطعی و قابل‌تکرار می‌خواهیم.

**نکته:** مدلی که استفاده می‌کنید باید از قابلیت **structured output** پشتیبانی کند.

حالا مدل را روی یک متن اجرا می‌کنیم:

```python
import pprint

text = """
Steven Paul Jobs (February 24, 1955 – October 5, 2011) was an American businessman,
inventor, and investor best known for co-founding the technology company Apple Inc.
Jobs was also the founder of NeXT and chairman and majority shareholder of Pixar. He was a
pioneer of the personal computer revolution of the 1970s and 1980s, along with his early business
partner and fellow Apple co-founder Steve Wozniak.
"""

model_output = structured_chat_model.invoke(text)

pprint.pp(model_output.model_dump())
```

خروجی:

```python
{'entities': [{'text': 'Steven Paul Jobs', 'label': 'PERSON'},
              {'text': 'February 24, 1955', 'label': 'DATE'},
              {'text': 'October 5, 2011', 'label': 'DATE'},
              {'text': 'American', 'label': 'ORG'},
              {'text': 'Apple Inc.', 'label': 'ORG'},
              {'text': 'NeXT', 'label': 'ORG'},
              {'text': 'Pixar', 'label': 'ORG'},
              {'text': 'Steve Wozniak', 'label': 'PERSON'},
              {'text': '1970s', 'label': 'DATE'},
              {'text': '1980s', 'label': 'DATE'}]}
```

در این مرحله مطمئن هستیم که خروجی، دقیقاً در قالب JSON مورد انتظار ما قرار دارد و می‌توانیم بدون نگرانی آن را در مراحل بعدی پردازش کنیم.

---

# تولید مجموعه‌دادهٔ جوک با LLM

به‌عنوان یک مثال دیگر، فرض کنید می‌خواهیم با کمک LLM یک مجموعه‌داده از جوک‌ها تولید کنیم. ابتدا شِمای داده را تعریف می‌کنیم:

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model
import pprint

class Joke(BaseModel):
    setup: str = Field(description="Setup of the joke")
    punchline: str = Field(description="Punchline of the joke")

class Jokes(BaseModel):
    joke: list[Joke]
```

سپس مدل را مقداردهی و شِمای خروجی را به آن اعمال می‌کنیم:

```python
chat_model = init_chat_model(
    model=MODEL_NAME,
    temperature=0.8,
)

structured_chat_model = chat_model.with_structured_output(Jokes)

prompt = "Give me three funny jokes."
model_output = structured_chat_model.invoke(prompt)

pprint.pp(model_output.model_dump())
```

خروجی:

```python
{'joke': [{'setup': "Why don't scientists trust atoms?",
           'punchline': 'Because they make up everything!'},
          {'setup': 'I used to hate facial hair...',
           'punchline': 'But then it grew on me.'},
          {'setup': 'Why did the scarecrow win an award?',
           'punchline': 'Because he was outstanding in his field!'}]}
```

در این سناریو، بدون نیاز به پارس دستی متن یا نوشتن کد، مستقیماً یک مجموعه‌دادهٔ تمیز و ساخت‌یافته دریافت کرده‌ایم که می‌تواند برای کاربردهای آینده استفاده شود.
