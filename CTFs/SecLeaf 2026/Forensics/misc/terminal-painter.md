# Look at the Structure Writeup

## 🏷️ Challenge Description
> "The artist said: 'Don't look at the symbols. Look at the structure.' Maybe the terminal knows more than your eyes do."

We are given an ASCII-art style text file `art.txt` containing the following pattern:
```text
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@%%%%@@@@@@@@@@@@
@@@@@@@@@@%%%%%%%%%%@@@@@@@@@@
@@@@@@%%%%%%%%%%%%%%%%@@@@@@@@
@@@@%%%%%%%%%%%%%%%%%%%%@@@@@@
@@%%%%%%%%%%%%%%%%%%%%%%%%@@@@
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%@%%%%%%%%%%%%%%%
%%%%%%%%%%%%%@@@%%%%%%%%%%%%%%%
%%%%%%%%%%%%@@@@@%%%%%%%%%%%%%%
%%%%%%%%%%%@@@@@@@%%%%%%%%%%%%%
%%%%%%%%%%@@@@@@@@@%%%%%%%%%%%%
%%%%%%%%%@@@@@@@@@@@%%%%%%%%%%%
```

---

## 🔍 Initial Analysis
The hint immediately suggests:
1. Ignore the actual characters (`@`, `%`).
2. Focus strictly on spacing, symmetry, geometry, and structure.

---

## 🛠️ Step-by-Step Investigation

### Step 1: Inspect File
We verify that there are no hidden bytes, steganography payload, or metadata tricks:
```bash
cat -A art.txt
xxd art.txt
```

**Result:**
* No hidden Unicode characters.
* No invisible bytes.
* Only plain ASCII text with newline-separated rows.

This confirms the encoding is entirely visual and structural.

### Step 2: Run-Length Analysis
We analyze contiguous character groups using Python to check for patterns:
```python
for line in open("art.txt"):
    s = line.rstrip()
    runs = []
    cur = s[0]
    cnt = 1
    for c in s[1:]:
        if c == cur:
            cnt += 1
        else:
            runs.append(cnt)
            cur = c
            cnt = 1
    runs.append(cnt)
    print(runs)
```

**Output:**
```json
[30]
[14, 4, 12]
[10, 10, 10]
[6, 16, 8]
[4, 20, 6]
[2, 24, 4]
[30]
[32]
[15, 1, 15]
[13, 3, 15]
[12, 5, 14]
[11, 7, 13]
[10, 9, 12]
[9, 11, 11]
```

**Observations:**
* Highly symmetric structure.
* Deliberate centered expansion.
* The lower section grows using odd-width progression: `1, 3, 5, 7, 9, 11`.
* This points to a precise geometric construction rather than random ASCII art.

---

## 🎨 Step 3: Visual Rendering
We can invert the symbols using `sed` to better understand the geometry.

### Rendering `%` as filled pixels:
```bash
sed 's/@/ /g; s/%/#/g' art.txt
```

**Output:**
```text
              ####            
          ##########          
      ################        
    ####################      
  ########################    
##############################
################################
############### ###############
#############   ###############
############     ##############
###########       #############
##########         ############
#########           ###########
```

### Rendering `@` as filled pixels:
```bash
sed 's/%/ /g; s/@/#/g' art.txt
```

**Output:**
```text
##############################
##############    ############
##########          ##########
######                ########
####                    ######
##                        ####
                              
                                
               #               
             ###               
            #####              
           #######             
          #########            
         ###########           
```

**Visual Conclusion:** This reveals neat geometric boundaries forming symmetric triangles.

---

## 📐 Step 4: Extract Structural Edges
The core data is contained in the transitions between characters rather than the characters themselves. We extract the indexes where transitions occur:
```python
for line in open("art.txt"):
    s = line.rstrip()
    pts = []
    for i in range(1, len(s)):
        if s[i] != s[i-1]:
            pts.append(i)
    print(pts)
```

**Output:**
```json
[]
[14, 18]
[10, 20]
[6, 22]
[4, 24]
[2, 26]
[]
[]
[15, 16]
[13, 16]
[12, 17]
[11, 18]
[10, 19]
[9, 20]
```

These coordinates form perfect diagonal boundaries.

---

## 📍 Step 5: Plotting the Structural Points
Plotting only the transition coordinates gives the underlying skeleton of the artwork:
```python
pts = []
for y, line in enumerate(open("art.txt")):
    s = line.rstrip()
    for i in range(1, len(s)):
        if s[i] != s[i-1]:
            pts.append((i, y))

mx = max(x for x, y in pts)
my = max(y for x, y in pts)

for y in range(my+1):
    row = ""
    for x in range(mx+1):
        row += "#" if (x, y) in pts else " "
    print(row)
```

**Output:**
```text
              #   #        
          #         #      
      #               #    
    #                   #  
  #                       #
                           
                           
               ##          
             #  #          
            #    #         
           #      #        
          #        #       
         #          #      
```

---

## 🧠 Core Lesson
This challenge was designed as a visual misdirection, tempting players to search for:
* Steganography in the file payload
* Hidden bytes or metadata
* OCR/object guessing

Instead, the solution is purely structural, focusing on terminal geometry, coordinate extraction, and transition boundaries. The symbols themselves were irrelevant; the logic resides in the relationship and symmetry between them.

---

## 🛠️ Key Techniques Used
* ASCII raster analysis
* Run-length encoding inspection
* Transition-point extraction
* Terminal visualization
* Geometric & structural decomposition
