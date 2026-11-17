VECTOR CLOCK SIMULATOR - MINI PROJECT

REQUIREMENTS
- Python 3.9 or newer
- Flask
- VS Code or any Python editor
- Web browser

HOW TO RUN (Windows)
1. Extract the ZIP file.
2. Open the extracted folder in VS Code.
3. Open Terminal in VS Code.
4. Install Flask:
      py -m pip install -r requirements.txt
   If 'py' does not work, try:
      python -m pip install -r requirements.txt
5. Run:
      py app.py
   Or:
      python app.py
6. Open this address in Chrome:
      http://127.0.0.1:5000

HOW TO TEST
1. Create 3 processes.
2. Choose P1 and click Local Event.
3. Choose P1 as sender and P2 as receiver. Click Send Message.
4. Choose P2 as receiving process and receive Message #1.
5. Check the vector clocks and event history.
6. Create more events and compare any two events.

EXPECTED VECTOR CLOCK EXAMPLE
Initially:
P1 = [0, 0, 0]
P2 = [0, 0, 0]
P3 = [0, 0, 0]

After P1 local event:
P1 = [1, 0, 0]

After P1 sends a message to P2:
P1 = [2, 0, 0]

After P2 receives that message:
P2 = [2, 1, 0]

PROJECT CONCEPT
For vectors A and B:
- A happened before B if A[i] <= B[i] for every position and A[i] < B[i] for at least one position.
- If neither A happened before B nor B happened before A, the events are concurrent.

NOTE
This is an educational simulator. It models message passing and logical clocks in one browser session; it does not create real networked computers.
