# Retrival augmented generation (RAG)
**NB:** I am using minsearch module in place of Elasticsearch. Next implementation will use Elasticsearch.

## Groq API Key
 - Create an account on GroqCloud console: <https://console.groq.com/login>
 - Navigate to 'API Key' tab <https://console.groq.com/keys>  and click on ``Create API Key`` button
 - For basic setup of chat completion using 'Groq SDK', navigate here: <https://console.groq.com/docs/text-chat>
 - Initiate ``Groq client``:
   ```
      from groq import Groq

      api_key = os.getenv('GROQ_API_KEY')
      client = Groq(api_key=api_key)
  
   ```
 - For Groq OpenAI compatibility setup, navigate here: <https://console.groq.com/docs/openai>

## Dev Environment setup:
 - Set up Virtual Environment: ``python -m venv .groq-env``
 - Activate: ``source .groq-env/bin/activate``

 ## RAG Framework Implementation
  - Implement a ``search`` function that will browse related queries in the indexed knowledge base.
    ```
        def search(query):
            boost = {
            'question': 3.0,
            'section': 0.5,   
            }
            
            results = index.search(
                query = query,
                filter_dict = {'course': 'data-engineering-zoomcamp'},
                boost_dict = boost,
                num_results = 5
            )
        
            return results
    ```
 
  - Do some prompt engineering to build a prompt that will be passed to the LLM.
  - Implement a ``build_function`` function that will trigger the prompt when called.
  - The function takes in two parameters ``query`` and ``search_results``
    
    ```
      def build_prompt(query, search_results):
          prompt_template = """
          You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database. 
          Only use the facts from the CONTEXT when answering the QUESTION.
          If the CONTEXT doesn't contain answer, output NONE. 
          Do not Quote the CONTEXT in the answer.
      
          QUESTION: {question}
          CONTEXT: {context}
          """.strip()
          
          context = ""
          
          for doc in search_results:
              context = context + f"section:{doc['section']}\n question:{doc['question']}\n text:{doc['text']}\n\n"
          
          prompt = prompt_template.format(question=query, context=context).strip()
          return prompt
    ```

    - For chat completion, implement a ``llm`` function that will take ``prompt`` parameter and return an answer.
        ```
        def llm(prompt):
            response = client.chat.completions.create(
            messages=[
                {
                    "role": "system",
                    "content": prompt
                },
                {
                    "role": "user",
                    "content": query,
                }
            ],
        
            # The language model which will generate the completion.
            model="llama-3.3-70b-versatile",

            temperature=0.5,
            max_tokens=1024,
            top_p=1,
            stop=None,
            stream=False,
            )
        
            # Print the completion returned by the LLM.
            return response.choices[0].message.content
        ```
    - Implement ``rag()`` function to execute all functions,  ``search()``, ``build_prompt()`` and ``llm()``.
    - ``rag()`` takes in ``query`` parameter.
    - Query example: ``query = "The course has already started, can I still enroll?"``
       ```
        query = "The course has already started, can I still enroll?"

        def rag(query):
            search_results = search(query)
            prompt = build_prompt(query, search_results)
            answer = llm(prompt)
            return answer

       rag(query)
       ```
