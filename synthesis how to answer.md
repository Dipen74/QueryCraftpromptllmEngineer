### **synthesis how to answer**



def synthesize\_answer(question: str, df: pd.DataFrame, result\_limit: int = 50) -> str:

&#x20;   if df is None or df.empty:

&#x20;       result\_summary = "The query ran successfully but returned no rows."

&#x20;       total\_note = ""

&#x20;   else:

&#x20;       total\_rows = len(df)

&#x20;       preview\_rows = min(total\_rows, 30)

&#x20;       result\_summary = df.head(preview\_rows).to\_string(index=False)

&#x20;       total\_note = f"\\n\\nTOTAL ROWS RETURNED: {total\_rows}"

&#x20;       if total\_rows == result\_limit:

&#x20;           total\_note += (

&#x20;               f" (this equals the query's LIMIT of {result\_limit} — "

&#x20;               "there may be MORE matching rows beyond what was returned)"

&#x20;           )

&#x20;       elif total\_rows > preview\_rows:

&#x20;           total\_note += f" (only the first {preview\_rows} are shown in the preview below)"



&#x20;   prompt = f"""Question: "{question}"



Query result (preview):

{result\_summary}

{total\_note}



Answer the question in 1-3 plain-English sentences using ONLY the data shown above.



\- Always base any count on TOTAL ROWS RETURNED above — never on how many rows happen to

&#x20; be listed in the preview text.

\- If TOTAL ROWS RETURNED equals the query's limit, say the list may be incomplete rather

&#x20; than presenting it as exhaustive.

\- If the result is empty, say so plainly and suggest a likely reason without inventing data.

\- If multiple rows are clearly different people sharing the same name, list their id and

&#x20; department instead of picking one arbitrarily.

\- Never state a percentage or total as complete if the underlying data doesn't support that."""



&#x20;   response = client.models.generate\_content(model=MODEL, contents=prompt)

&#x20;   return response.text.strip()





