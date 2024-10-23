# Instruqt workshop style guide

## Text

<span style="color:#FF9E16">Orange text for something that should stand out - a focus point.</span>

<span style="color:#A5E9C5">'Magic mint' colour - for top level instructions</span>

<span style="color:#B4DDE6">Coloured text (powder blue).</span>

<span style="background-color:#119e6f;font-weight:bold;color:white">A green background with white text</span>


**<span style="color:#B4DDE6">Definiton:</span>**
What the definition is.

**<span style="color:#B4DDE6">Another definition:</span>**
More words about that thing.


## Buttons

Full width button:
[button label="White text on a denim background" block background="#FFFFFF" color="#1084BD"](url)

Button:
[button label="Very light blue/grey on a denim background" background="#F7F9FC" color="#1084BD"](url)


## Sections and breaks

🐱 Section header
===
The emoji helps break up the monotony of text.

---
This is another section

## Variables

Set the variable value in the setup script:
`agent variable set UI_SECRET $(cat /root/ui-secret.txt)`

Use the variable in the assignment:
`[[ Instruqt-Var key="UI_SECRET" hostname="c1-control" ]]`


## Additional text things

> [!IMPORTANT]
> Something important


> [!NOTE]
> A note.


> [!TIP]
> ℹ️ Tips are unsupported in Instruqt


## Instructions

# 📝 <span style="color:#A5E9C5">The overall goal of these instructions:</span>

**<span style="color:#B4DDE6">1. This is the first step</span>**

Instructions go here.

**<span style="color:#B4DDE6">2. This is the second step</span>**
In these instructions you might need to configure multiple things. 

Use emojis to help make a visually easy to follow list:

➕ add this

➕ and this

➕ and this


## Run a command

```yaml
kubectl apply -f - <<-EOF
yaml: goesHere
EOF
```

## Tables

The :----: indicates column width.

**Source** | **Destination** | **Result**
:-------: | :-------: | :-------:
First column | Second column | <span style="color:#FF2400">Red text</span>
