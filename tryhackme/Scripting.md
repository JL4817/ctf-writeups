This file has been encoded 50 times - write a script to retrieve the flag. Here is the general process to do this:

    read input from the file
    use function to decode the file
    do process in a loop

Try do this in both Bash and Python!
Answer the questions below

What is the final string?

import base64

# Read the encoded file
with open("b64.txt", "rb") as file:
    data = file.read()

# Decode 50 times
for i in range(50):
    data = base64.b64decode(data)

# Print the final string
print(data.decode("utf-8"))
