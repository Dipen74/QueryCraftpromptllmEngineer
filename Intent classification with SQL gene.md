### **Intent classification with SQL generator**



import json



def classify\_question(question: str, schema\_text: str) -> dict:

&#x20;   prompt = CLASSIFY\_PROMPT\_TEMPLATE.format(schema=schema\_text, question=question)

&#x20;   try:

&#x20;       response = client.models.generate\_content(model=MODEL, contents=prompt)

&#x20;       raw = response.text.replace("```json", "").replace("```", "").strip()

&#x20;       result = json.loads(raw)

&#x20;       if result.get("label") not in ("ANSWERABLE", "AMBIGUOUS", "OUT\_OF\_SCOPE"):

&#x20;           raise ValueError("unexpected label")

&#x20;       return result

&#x20;   except Exception:

&#x20;       return {"label": "ANSWERABLE", "reason": "classifier fallback", "clarifying\_question": ""}





def generate\_sql(question: str, schema\_text: str, error\_context: str = None) -> str:

&#x20;   prompt = f"""{SYSTEM\_PROMPT}



{FEW\_SHOT\_EXAMPLES}



DATABASE SCHEMA CONTEXT:

{schema\_text}



USER QUESTION:

"{question}"

"""



&#x20;   if error\_context:

&#x20;       prompt += f"""

PREVIOUS ATTEMPT FAILED:

The previous SQL query failed during execution/validation with this error:

{error\_context}



Review the error and the schema, fix the mistake, and return a corrected single SQLite SELECT/WITH query.

"""



&#x20;   response = client.models.generate\_content(model=MODEL, contents=prompt)

&#x20;   clean\_sql = response.text.replace("```sql", "").replace("```", "").strip()



&#x20;   if not is\_safe\_select(clean\_sql):

&#x20;       raise ValueError(

&#x20;           "Generated query failed the safety check — only a single read-only "

&#x20;           "SELECT/WITH statement is allowed (no writes, no stacked statements)."

&#x20;       )



&#x20;   return clean\_sql



