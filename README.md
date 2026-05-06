# QualiShield
Automated redaction tool using Python. 
import re

def deidentify_text(text):
    # --- REGEX REDACTIONS ---

    # Dates (MM/DD/YYYY or similar)
    text = re.sub(r'\b\d{1,2}[/-]\d{1,2}[/-]\d{2,4}\b', '[DATE]', text)

    # Emails
    text = re.sub(r'\b[\w\.-]+@[\w\.-]+\.\w+\b', '[EMAIL]', text)

    # Phone numbers (123-456-7890, 123.456.7890, 1234567890)
    text = re.sub(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b', '[PHONE]', text)

    # Locations
    locations = ["Atlanta", "Georgia", "New York", "California"]
    for loc in locations:
        text = re.sub(rf'\b{loc}\b', '[LOCATION]', text)

    # --- NAME REDACTION ---
    words = text.split()
    new_words = []

    common_words = {
        "The", "A", "An", "This", "That", "His", "Her",
        "He", "She", "It", "They", "We", "I",
        "And", "But", "Or", "If", "In", "On", "At"
    }

    for word in words:
        # Skip already-redacted tokens
        if "[" in word:
            new_words.append(word)
            continue

        # Detect capitalized words that look like names
        if word.istitle() and word not in common_words:
            new_words.append("[NAME]")
        else:
            new_words.append(word)

    return " ".join(new_words)


print("QualiShield - De-Identification Tool")

# --- AUTO-LOAD sample.txt OR MANUAL INPUT ---
choice = input("\nType 'file' to load sample.txt or 'text' to paste text: ").strip().lower()

if choice == "file":
    filename = "sample.txt"
    try:
        with open(filename, "r", encoding="utf-8") as f:
            text = f.read()
        print(f"\nLoaded {filename}")
    except FileNotFoundError:
        print("\nERROR: sample.txt not found in this folder.")
        exit()

else:
    text = input("\nPaste transcript:\n")

# Process text
result = deidentify_text(text)

# Print result
print("\n--- Redacted Transcript ---")
print(result)

# Save to file
with open("redacted_output.txt", "w", encoding="utf-8") as file:
    file.write(result)

print("\nSaved to redacted_output.txt")

