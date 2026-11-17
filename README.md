from flask import Flask, render_template, request, redirect, url_for, session, flash
from copy import deepcopy

app = Flask(__name__)
app.secret_key = "vector-clock-mini-project-key"

def initialize(count=3):
    session["count"] = count
    session["clocks"] = [[0] * count for _ in range(count)]
    session["events"] = []
    session["messages"] = []
    session["next_event"] = 1

def add_event(process, action, clock):
    events = session.get("events", [])
    events.append({
        "id": session["next_event"],
        "process": f"P{process + 1}",
        "action": action,
        "clock": list(clock)
    })
    session["events"] = events
    session["next_event"] += 1

@app.route("/", methods=["GET", "POST"])
def index():
    if "clocks" not in session:
        initialize()

    if request.method == "POST":
        action = request.form.get("action")
        clocks = session["clocks"]
        count = session["count"]

        if action == "reset":
            try:
                n = max(2, min(5, int(request.form.get("count", 3))))
            except ValueError:
                n = 3
            initialize(n)
            flash(f"Created {n} processes.")
            return redirect(url_for("index"))

        try:
            p = int(request.form.get("process", 0))
            if p < 0 or p >= count:
                raise ValueError
        except ValueError:
            flash("Select a valid process.")
            return redirect(url_for("index"))

        if action == "local":
            clocks[p][p] += 1
            add_event(p, "Local event", clocks[p])

        elif action == "send":
            try:
                target = int(request.form.get("target"))
                if target < 0 or target >= count or target == p:
                    raise ValueError
            except (ValueError, TypeError):
                flash("Choose a different valid receiver.")
                return redirect(url_for("index"))

            clocks[p][p] += 1
            message = {
                "id": len(session["messages"]) + 1,
                "sender": p,
                "receiver": target,
                "clock": list(clocks[p])
            }
            messages = session["messages"]
            messages.append(message)
            session["messages"] = messages
            add_event(p, f"Sent message to P{target + 1}", clocks[p])

        elif action == "receive":
            try:
                message_id = int(request.form.get("message_id"))
                messages = session["messages"]
                message = next(m for m in messages if m["id"] == message_id)
                if message["receiver"] != p or message.get("received"):
                    raise ValueError
            except (ValueError, StopIteration, TypeError):
                flash("Select a pending message addressed to this process.")
                return redirect(url_for("index"))

            for i in range(count):
                clocks[p][i] = max(clocks[p][i], message["clock"][i])
            clocks[p][p] += 1
            messages = session["messages"]
            for m in messages:
                if m["id"] == message_id:
                    m["received"] = True
            session["messages"] = messages
            add_event(p, f"Received message #{message_id}", clocks[p])

        session["clocks"] = clocks
        return redirect(url_for("index"))

    return render_template(
        "index.html",
        clocks=session["clocks"],
        count=session["count"],
        events=session["events"],
        messages=session["messages"]
    )

@app.route("/compare", methods=["POST"])
def compare():
    events = session.get("events", [])
    try:
        first_id = int(request.form["first"])
        second_id = int(request.form["second"])
        first = next(e for e in events if e["id"] == first_id)
        second = next(e for e in events if e["id"] == second_id)
    except (ValueError, KeyError, StopIteration):
        flash("Choose two valid events.")
        return redirect(url_for("index"))

    a, b = first["clock"], second["clock"]
    a_before_b = all(x <= y for x, y in zip(a, b)) and any(x < y for x, y in zip(a, b))
    b_before_a = all(x <= y for x, y in zip(b, a)) and any(x < y for x, y in zip(b, a))

    if a_before_b:
        result = f"Event {first_id} happened BEFORE event {second_id} (causal relationship)."
    elif b_before_a:
        result = f"Event {second_id} happened BEFORE event {first_id} (causal relationship)."
    else:
        result = f"Events {first_id} and {second_id} are CONCURRENT."

    session["comparison"] = result
    return redirect(url_for("index"))

@app.context_processor
def inject_comparison():
    return {"comparison": session.get("comparison", "")}

if __name__ == "__main__":
    app.run(debug=True)
