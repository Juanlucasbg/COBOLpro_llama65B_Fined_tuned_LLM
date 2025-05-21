---
library_name: transformers
license: apache-2.0
---

# Model Card for mAInframer-1

LLM for the COBOL programming language.

## Model Details

### Model Description

mAInframer is a series of models with different parameter counts: 7b, 13b, 34b.

These models are pretrained on code (CodeLlama base models) and fine-tuned on COBOL code.


## Uses

This model can be used for code completion and fill-in-the middle (COBOL).

## How to Get Started with the Model

Model loading:

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
model = AutoModelForCausalLM.from_pretrained("bloopai/mAInframer-7b", device_map="cuda", torch_dtype=torch.bfloat16)
tokenizer = AutoTokenizer.from_pretrained("codellama/CodeLlama-7b-hf")
```

### Completion:

```python
prompt = '''       IDENTIFICATION DIVISION.
       PROGRAM-ID.  SUM-OF-CUBES.
       ENVIRONMENT DIVISION.
       
       INPUT-OUTPUT SECTION.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       
       01 STEP         PIC S9(10).
       01 CUBE         PIC 9(7).
       01 CUBE-SUM     PIC 9(7) VALUE 0.

       LINKAGE SECTION.

       01 LINKED-ITEMS.
           05 L-MAX-STEP PIC S9(10).
           05 RESULT PIC S9(10).

      * 
      * Given an integer number, return the sum of the of all the integers below it.
      * 
      * Example:
      * 
      * sum_of_cubes(3) == 1**3 + 2**3 == 9
      * sum_of_cubes(5) == 100
      *  

      * Store the result in the RESULT variable and mark the end of your program with END PROGRAM
'''
inputs = tokenizer.encode(prompt, return_tensors="pt", add_special_tokens=False).to("cuda")
outputs = model.generate(inputs, max_new_tokens=250, use_cache=True, do_sample=False, repetition_penalty=1.1)
print(self.tokenizer.decode(outputs[0]))
```

Result:
```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID.  SUM-OF-CUBES.
       ENVIRONMENT DIVISION.
       
       INPUT-OUTPUT SECTION.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       
       01 STEP         PIC S9(10).
       01 CUBE         PIC 9(7).
       01 CUBE-SUM     PIC 9(7) VALUE 0.

       LINKAGE SECTION.

       01 LINKED-ITEMS.
           05 L-MAX-STEP PIC S9(10).
           05 RESULT PIC S9(10).

      * 
      * Given an integer number, return the sum of the of all the integers below it.
      * 
      * Example:
      * 
      * sum_of_cubes(3) == 1**3 + 2**3 == 9
      * sum_of_cubes(5) == 100
      *  

      * Store the result in the RESULT variable and mark the end of your program with END PROGRAM

       PROCEDURE DIVISION USING LINKED-ITEMS.
           MOVE L-MAX-STEP TO STEP.
           PERFORM VARYING STEP FROM 1 BY 1 UNTIL STEP > L-MAX-STEP
               COMPUTE CUBE = STEP ** 3
               ADD CUBE TO CUBE-SUM
           END-PERFORM.
           DISPLAY CUBE-SUM.
           MOVE CUBE-SUM TO RESULT.
           GOBACK.
           
       END PROGRAM SUM-OF-CUBES.
```

### Infilling

Follow the format: `<PRE>prefix<SUF>suffix<MID>`

To complete `PROCEDURE DIVISION` and infill `WORKING STORAGE SECTION.` to solve [COBOLEval](https://github.com/BloopAI/COBOLEval) problems:

```python
prompt = '''<PRE>       IDENTIFICATION DIVISION.
       PROGRAM-ID.  SUM-OF-CUBES.
       ENVIRONMENT DIVISION.
       
       INPUT-OUTPUT SECTION.

       DATA DIVISION.<SUF>

       LINKAGE SECTION.

       01 LINKED-ITEMS.
           05 L-MAX-STEP PIC S9(10).
           05 RESULT PIC S9(10).

      * 
      * Given an integer number, return the sum of the of all the integers below it.
      * 
      * Example:
      * 
      * sum_of_cubes(3) == 1**3 + 2**3 == 9
      * sum_of_cubes(5) == 100
      *  

      * Store the result in the RESULT variable and mark the end of your program with END PROGRAM'''
```

Result:
```cobol
<PRE>       IDENTIFICATION DIVISION.
       PROGRAM-ID. MAX-ELEMENT.

       ENVIRONMENT DIVISION.
       
       INPUT-OUTPUT SECTION.

       DATA DIVISION.<SUF>
       LINKAGE SECTION.

       01 LINKED-ITEMS.
           05 L-L OCCURS 100 TIMES INDEXED BY NI PIC S9(10).
           05 RESULT PIC S9(10).

      * Return maximum element in the list.
      * >>> max_element([1, 2, 3])
      * 3
      * >>> max_element([5, 3, -5, 2, -3, 3, 9, 0, 123, 1, -10])
      * 123
      * 

      * Store the result in the RESULT variable and mark the end of your program with END PROGRAM

       PROCEDURE DIVISION USING LINKED-ITEMS.
           MOVE ZERO TO WS-MAX-VALUE.
           PERFORM VARYING NI FROM 1 BY 1 UNTIL NI > 100
               IF L-L (NI) > WS-MAX-VALUE THEN
                   MOVE L-L (NI) TO WS-MAX-VALUE
               END-IF
           END-PERFORM.
           DISPLAY 'THE MAXIMUM ELEMENT IS: ' WS-MAX-VALUE.
           MOVE WS-MAX-VALUE TO RESULT.
           GOBACK.
           
       END PROGRAM MAX-ELEMENT.

<MID>
       WORKING-STORAGE SECTION.
       
       01 WS-MAX-VALUE PIC S9(10) VALUE ZERO.

```

## Training Details

Base model: CodeLlama
Finetuning type: LoRA

### Metrics

[COBOLEval](https://github.com/BloopAI/COBOLEval) is an adaptation of HumanEval where the problems are translated to COBOL.

| **Model**            | CobolEval (pass@1) | 
|----------------------|--------------------|
| **mAInframer-7b**    | 6.16   | 
| **mAInframer-13b**   | 8.90   |
| **mAInframer-34b**   | 10.27  |


## Citation 

[Blog post]() 


## Model Card Contact

[More Information Needed]