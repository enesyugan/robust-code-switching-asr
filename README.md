# Robust Code-Switching ASR Adaptation

Official implementation of:

> **Adding Robust Code-Switching Capabilities to High Performance Multilingual ASR**

## Related Repository

The full training and adaptation framework used in our experiments is available in the companion repository:

**[continual-asr](https://github.com/enesyugan/continual-asr)**

That repository contains:

- Whisper continual learning and adaptation code
- Qwen2-Audio adaptation for code-switching ASR
- LoRA, Bayesian LoRA (BLoRA), PiSSA, OLoRA, EVA, RS-LoRA, and DoRA
- Batch ensembles and weight factorization methods
- Training, decoding, and evaluation pipelines

This repository focuses on the prompting strategies, annotation guidelines, and experimental resources used for code-switching data generation and evaluation.

This repository contains the code, prompts, annotation guidelines, and experimental resources accompanying our work on **code-switching automatic speech recognition (ASR)** adaptation for **strong multilingual speech recognition models**.

---

## Repository Contents

This repository includes:

- The LLM prompting strategy for controlled German–English code-switch generation

- Annotation guidelines for identifying English code-switches in German–English speech

- Resources used for synthetic code-switching data generation

- Experimental materials accompanying the paper

For model training and adaptation code, see the companion repository:

**[continual-asr](https://github.com/enesyugan/continual-asr)**

---

## 1. LLM-Based Synthetic Code-Switch Generation

To generate controlled German–English code-switched sentences, we used GPT-4o as the underlying language model.

**Model configuration:**

- Model: GPT-4o  
- Temperature: 0.3  
- Decoding: deterministic-leaning setup to encourage morphosyntactic stability and reproducibility  

The prompt was designed to enforce:

- Strict single-word substitution  
- German matrix language  
- English base-form insertion  
- Morphological integration into German  
- Preservation of lexical meaning, valency, and register  
- High grammatical and idiomatic acceptability  

### Prompt Used for Code-Switch Generation

```text
Act as a linguist proficient in German and English.
Change the following German sentence by replacing exactly one adjective, verb, or noun with an English equivalent and adapt the sentence so that it becomes a code-switched sentence.
Use German as the matrix language, and insert the English base form (e.g., “increase” for “steigend”) with the following constraints:

1. Derivational and Inflectional Morphology
Apply appropriate German morphology to the English base so it fits seamlessly:
    • Verbs:
        - Infinitives: append the German infinitive suffix -en to the English base (→ “manage” becomes “managen”).
        - Finite forms: conjugate the English base with German person/number/tense endings 
          (e.g. “manage” → “managst” for 2 SG, “manage” → “managet” for 3 SG).
        - Participles: if the original is a participle (“ge-…”, “…-t”), form the English participle 
          (e.g. “uploaded”) and render it as a German participle (“geuploadet”).
    • Nouns:
        - Apply German gender, article, case, and plural forms (e.g., “ein Tool”, “der Managerin”), 
          and ensure that all nouns begin with a capital letter in accordance with German orthography.
        - For nominalized infinitives (e.g., “das Waschen”):
            1. Take the English base (“wash”).
            2. Add the German nominalizing suffix -en → “Washen”.
            3. Inflect that “Washen” for the same gender/case/number as the original 
               (“das Washen”, neuter nominative singular).
    • Adjectives:
        - Decline as needed for gender, number, and case.

2. Syntactic and Semantic Match
Ensure that the substituted word has the same word class, reflexivity, valency, and lexical meaning 
as the original word.
If the original verb is reflexive, the English replacement must also be reflexive and morphologically 
adapted accordingly.
For example, "ich ernähre mich" should be substituted with 
"ich §§dietiere§§ mich" and not with "ich §§manage§§ mich".

3. Register Compatibility
Match the tone, style, and formality level of the surrounding sentence. 
Do not insert slang terms into formal contexts.

4. Compound Handling
If the substitution occurs within a compound noun, obey German compounding rules:
    • Use correct constituent order (“Geologie-Background” not “Background-Geologie”)
    • Add linking elements or hyphens where needed

5. Idiomaticity and Acceptability
If a direct substitution leads to an unidiomatic or grammatically incorrect result, 
rephrase the sentence or choose a different eligible word for substitution.

6. Fallback Strategy
If no morphosyntactically and idiomatically valid substitution is possible, 
retry with a different part of speech in the sentence.

Output: Only print the final grammatically correct, idiomatic, and natural-sounding 
code-switched sentence. Wrap the inserted English-derived word (after applying German 
morphology) in §§ ...§§. Mark only the substituted word, not surrounding context. 
No explanations, no comments.
```

## 2. Code-Switching Annotation Guidelines

## Matrix Language

The matrix language of the corpus is **German**.  
A token is annotated as *code-switched (CS)* if it represents an insertion from a non-German language into an otherwise German syntactic frame.

---

## A Token Is Annotated as Code-Switched (CS) If:

### 1. Lexical Insertion from a Living Foreign Language  
The token belongs to a living non-German language (e.g., English) and a standard German equivalent exists.

**Examples:**  
- `because` → *weil*  
- `runway` → *Landebahn*  
- `Portuguese` → *Portugiesisch*  
- `losing streak` → *Niederlagenserie*

---

### 2. English Function Words Occur  
Articles, conjunctions, pronouns, auxiliaries, etc.

**Examples:**  
- `if`, `because`, `as`, `the`, `a`, `will`, `can`

Function words are always marked as CS.

---

### 3. English Phrases or Technical Terms Are Inserted  
Multi-word expressions are marked at the token level.

**Examples:**  
- `hostile fire`  
- `Romance language`  
- `runway overrun`

---

### 4. English Possessive Morphology Appears  
Possessive ’s constructions are marked as CS.

**Examples:**  
- `Australia’s` → *Australiens*  
- `men’s` → *Männer-*

---

### 5. English Numerals Are Used  
**Example:**  
- `seventeen` → *siebzehn*

---

### 6. Hybrid Forms with English Lexical Base  
German inflection applied to an English stem.

**Examples:**  
- `moven`  
- `breachen`  
- `downloaden`

---

### 7. English Pronunciation of Acronyms (Audio Condition)  
If clearly pronounced using English phonology.

**Example:**  
- `USA` pronounced “you-ess-ay” → CS  

---

## A Token Is NOT Annotated as Code-Switched If:

### 1. Proper Names  
Persons, locations, institutions, teams.

**Examples:**  
- `Hong Kong`  
- `T-Rex`  
- `Sachin Tendulkar`

---

### 2. Internationally Standardized Codes or Technical Labels  
ISO currency codes, medical acronyms, etc.

**Examples:**  
- `USD`, `GBP`, `FKP`  
- `XDR-TB`

(Unless clearly pronounced in English in audio.)

---

### 3. Fully Integrated Loanwords in German  
Entries established in standard German and morphologically integrated.

**Examples:**  
- `Team / Teams`  
- `Blog / Blogs`  
- `Crash`

However, the same word inside a fully English phrase (e.g., `invisible team`) is marked as CS.

---

### 4. Citation Languages (Etymological or Scholarly Use)  
Latin, Ancient Greek, etc., when used as citation forms rather than communicative switches.

**Examples:**  
- `civitas`  
- `civilis`

---

## Operational Principle

Code-switching is defined as an **active insertion of a foreign communicative code** into a German syntactic structure.

Not considered code-switching:
- Proper names  
- International symbols  
- Established loanwords  
- Etymological citations  

---

## Citation

Please cite the relevant paper:

```bibtex

@article{ugan2026adding,
  title={Adding Robust Code-Switching Capabilities to High Performance Multilingual ASR},
  author={Ugan, Enes Yavuz and Waibel, Alexander},
  journal={arXiv preprint arXiv:2606.21990},
  year={2026}
}
```
