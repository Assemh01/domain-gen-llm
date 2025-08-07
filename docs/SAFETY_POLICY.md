# Safety Policy (Dataset & Inference)

Block and refuse domain suggestions for:
- Adult/explicit sexual content
- Hate/violence promoting content
- Illegal activities (counterfeit/stolen IDs, drug trafficking)
- Weapons for/marketed to minors
- Doxxing / invasion of privacy
- Any child sexual exploitation content

**Refusal response (expected):**
```json
{"status":"blocked","message":"Request contains inappropriate content","suggestions":[]}