import tkinter as tk
from tkinter import ttk, messagebox


# ----------------------------- ALGORITHM ---------------------------------

def parse_matrix(text, rows, cols, matrix_name):
    """Read a matrix containing one process per line."""
    lines = [line.strip() for line in text.strip().splitlines() if line.strip()]
    if len(lines) != rows:
        raise ValueError(
            f"{matrix_name} must contain exactly {rows} rows; found {len(lines)}."
        )

    matrix = []
    for row_number, line in enumerate(lines, start=1):
        parts = line.replace(",", " ").split()
        if len(parts) != cols:
            raise ValueError(
                f"{matrix_name}, row {row_number}: expected {cols} values; "
                f"found {len(parts)}."
            )
        try:
            values = [int(value) for value in parts]
        except ValueError:
            raise ValueError(
                f"{matrix_name}, row {row_number}: enter whole numbers only."
            )
        if any(value < 0 for value in values):
            raise ValueError(f"{matrix_name}, row {row_number}: values cannot be negative.")
        matrix.append(values)
    return matrix


def validate_state(total, allocation, maximum):
    """Check dimensions and ensure allocations do not exceed declared values."""
    if not total or any(value < 0 for value in total):
        raise ValueError("Total vector must contain non-negative values.")

    process_count = len(allocation)
    resource_count = len(total)

    if process_count == 0:
        raise ValueError("Enter at least one process.")
    if len(maximum) != process_count:
        raise ValueError("Allocation and Max must have the same number of rows.")

    for matrix_name, matrix in (("Allocation", allocation), ("Max", maximum)):
        if any(len(row) != resource_count for row in matrix):
            raise ValueError(
                f"{matrix_name} must have {resource_count} columns, "
                "matching the resource types."
            )

    for i in range(process_count):
        for j in range(resource_count):
            if allocation[i][j] > maximum[i][j]:
                raise ValueError(
                    f"Invalid state: Allocation P{i}, resource R{j}, is greater than Max."
                )

    allocated_totals = [
        sum(allocation[i][j] for i in range(process_count))
        for j in range(resource_count)
    ]
    for j in range(resource_count):
        if allocated_totals[j] > total[j]:
            raise ValueError(
                f"Invalid state: allocated amount of resource R{j} exceeds Total."
            )


def calculate_need(maximum, allocation):
    return [
        [maximum[i][j] - allocation[i][j] for j in range(len(maximum[i]))]
        for i in range(len(maximum))
    ]


def calculate_available(total, allocation):
    return [
        total[j] - sum(row[j] for row in allocation)
        for j in range(len(total))
    ]


def less_equal(left, right):
    return all(a <= b for a, b in zip(left, right))


def safety_algorithm(total, allocation, maximum):
    """Return whether the state is safe, plus its trace and computed values."""
    validate_state(total, allocation, maximum)
    need = calculate_need(maximum, allocation)
    available = calculate_available(total, allocation)
    work = available[:]
    finish = [False] * len(allocation)
    sequence = []
    trace = [
        f"Initial Work = Available = {work}",
        f"Initial Finish = {finish}",
    ]

    while len(sequence) < len(allocation):
        found_process = False

        for i in range(len(allocation)):
            if finish[i]:
                continue

            can_finish = less_equal(need[i], work)
            trace.append(
                f"Check P{i}: Need {need[i]} <= Work {work} -> "
                f"{'TRUE' if can_finish else 'FALSE'}"
            )

            if can_finish:
                old_work = work[:]
                work = [
                    work[j] + allocation[i][j]
                    for j in range(len(work))
                ]
                finish[i] = True
                sequence.append(f"P{i}")
                trace.append(
                    f"  P{i} can finish. Work = {old_work} + "
                    f"Allocation[P{i}] {allocation[i]} = {work}"
                )
                trace.append(f"  Finish = {finish}")
                found_process = True
                break

        if not found_process:
            trace.append("No unfinished process has Need <= Work.")
            break

    safe = all(finish)
    if safe:
        trace.extend([
            "RESULT: SAFE STATE",
            "Safe sequence: " + " -> ".join(sequence),
        ])
    else:
        unfinished = [
            f"P{i}" for i, completed in enumerate(finish) if not completed
        ]
        trace.extend([
            "RESULT: UNSAFE STATE",
            "Processes that could not finish: " + ", ".join(unfinished),
        ])

    return {
        "safe": safe,
        "sequence": sequence,
        "need": need,
        "available": available,
        "trace": trace,
        "finish": finish,
        "final_work": work,
    }


def check_resource_request(total, allocation, maximum, process_index, request):
    """Tentatively allocate a request and grant it only if the state stays safe."""
    validate_state(total, allocation, maximum)
    process_count = len(allocation)
    resource_count = len(total)

    if not 0 <= process_index < process_count:
        raise ValueError(f"Choose a process index from 0 to {process_count - 1}.")
    if len(request) != resource_count:
        raise ValueError(f"Request vector must contain exactly {resource_count} values.")
    if any(value < 0 for value in request):
        raise ValueError("Request values cannot be negative.")

    need = calculate_need(maximum, allocation)
    available = calculate_available(total, allocation)

    trace = [
        f"Request by P{process_index} = {request}",
        f"Request <= Need {need[process_index]}: "
        f"{less_equal(request, need[process_index])}",
    ]

    if not less_equal(request, need[process_index]):
        trace.append(
            "DECISION: WAIT — request exceeds the process's declared Need."
        )
        return False, trace

    trace.append(
        f"Request <= Available {available}: {less_equal(request, available)}"
    )
    if not less_equal(request, available):
        trace.append("DECISION: WAIT — resources are not currently available.")
        return False, trace

    tentative_allocation = [row[:] for row in allocation]
    tentative_allocation[process_index] = [
        tentative_allocation[process_index][j] + request[j]
        for j in range(resource_count)
    ]

    trace.append("Tentative allocation made. Checking the resulting state...")
    result = safety_algorithm(total[:], tentative_allocation, [row[:] for row in maximum])
    trace.extend(result["trace"])

    if result["safe"]:
        trace.append("DECISION: GRANT — the tentative state is safe.")
        return True, trace

    trace.append("DECISION: WAIT — granting the request would make the state unsafe.")
    return False, trace


# ------------------------------- GUI -------------------------------------

class BankersGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Banker's Algorithm | PT 101")
        self.root.geometry("1180x820")
        self.root.minsize(900, 650)
        self.root.configure(bg="#f3f6fb")

        self.process_var = tk.StringVar(value="5")
        self.resource_var = tk.StringVar(value="3")
        self.total_var = tk.StringVar(value="10 5 7")
        self.request_process_var = tk.StringVar(value="1")
        self.request_var = tk.StringVar(value="1 0 2")

        self._configure_styles()
        self._build_interface()
        self.load_sample()

    def _configure_styles(self):
        style = ttk.Style()
        try:
            style.theme_use("clam")
        except tk.TclError:
            pass

        style.configure("TFrame", background="#f3f6fb")
        style.configure("Card.TFrame", background="white")
        style.configure(
            "TLabel", background="#f3f6fb", foreground="#172033",
            font=("Segoe UI", 10)
        )
        style.configure(
            "Title.TLabel", background="#f3f6fb", foreground="#13294b",
            font=("Segoe UI", 19, "bold")
        )
        style.configure(
            "Sub.TLabel", background="#f3f6fb", foreground="#52627a",
            font=("Segoe UI", 10)
        )
        style.configure(
            "CardTitle.TLabel", background="white", foreground="#17365d",
            font=("Segoe UI", 11, "bold")
        )
        style.configure("TButton", font=("Segoe UI", 9, "bold"), padding=7)
        style.configure(
            "Accent.TButton", background="#1769aa", foreground="white"
        )
        style.map("Accent.TButton", background=[("active", "#0f568f")])
        style.configure("TEntry", padding=5)

    def _build_interface(self):
        header = ttk.Frame(self.root, padding=(18, 14, 18, 7))
        header.pack(fill="x")
        ttk.Label(
            header, text="BANKER'S ALGORITHM SIMULATOR",
            style="Title.TLabel"
        ).pack(anchor="w")
        ttk.Label(
            header, text="PT 101 • Topic 4: Deadlock Avoidance",
            style="Sub.TLabel"
        ).pack(anchor="w", pady=(3, 0))

        body = ttk.Frame(self.root, padding=(18, 7, 18, 12))
        body.pack(fill="both", expand=True)

        # LEFT SIDE: the entire input card is inside a Canvas, so the
        # Optional: Resource Request section can be reached by scrolling.
        left_container = ttk.Frame(body)
        left_container.pack(side="left", fill="y", padx=(0, 12))

        self.left_canvas = tk.Canvas(
            left_container, width=390, height=620,
            background="white", highlightthickness=0, borderwidth=0
        )
        left_scrollbar = ttk.Scrollbar(
            left_container, orient="vertical",
            command=self.left_canvas.yview
        )
        self.left_canvas.configure(yscrollcommand=left_scrollbar.set)
        left_scrollbar.pack(side="right", fill="y")
        self.left_canvas.pack(side="left", fill="both", expand=True)

        self.input_card = ttk.Frame(
            self.left_canvas, style="Card.TFrame", padding=14
        )
        self.left_window = self.left_canvas.create_window(
            (0, 0), window=self.input_card, anchor="nw"
        )
        self.input_card.bind("<Configure>", self._update_left_scroll)
        self.left_canvas.bind("<Configure>", self._resize_left_content)

        self._build_input_controls()

        # RIGHT SIDE: results have their own scrollbar.
        right = ttk.Frame(body, style="Card.TFrame", padding=12)
        right.pack(side="left", fill="both", expand=True)

        ttk.Label(
            right, text="COMPUTATION & RESULTS",
            style="CardTitle.TLabel"
        ).pack(anchor="w")
        ttk.Label(
            right,
            text="The trace shows each Need-versus-Work comparison and update.",
            background="white", foreground="#3344a5"
        ).pack(anchor="w", pady=(3, 8))

        result_frame = ttk.Frame(right, style="Card.TFrame")
        result_frame.pack(fill="both", expand=True)

        self.output = tk.Text(
            result_frame, font=("Consolas", 9), wrap="word",
            bg="#101827", fg="#e7edf7", insertbackground="white",
            relief="flat", padx=10, pady=10
        )
        result_scrollbar = ttk.Scrollbar(
            result_frame, orient="vertical", command=self.output.yview
        )
        self.output.configure(yscrollcommand=result_scrollbar.set)
        self.output.pack(side="left", fill="both", expand=True)
        result_scrollbar.pack(side="right", fill="y")

        self.output.tag_configure(
            "good", foreground="#75e0a0", font=("Consolas", 9, "bold")
        )
        self.output.tag_configure(
            "bad", foreground="#ff9b9b", font=("Consolas", 9, "bold")
        )
        self.output.tag_configure(
            "heading", foreground="#8ecbff", font=("Consolas", 9, "bold")
        )

        self._enable_mousewheel(self.left_canvas)
        self._enable_mousewheel(self.output)

        footer = ttk.Label(
            self.root,
            text="Use the scrollbar or mouse wheel over a panel to scroll.",
            background="#f3f6fb", foreground="#65758b"
        )
        footer.pack(anchor="w", padx=20, pady=(0, 8))

    def _build_input_controls(self):
        card = self.input_card

        ttk.Label(
            card, text="INPUT DATA", style="CardTitle.TLabel"
        ).pack(anchor="w")

        dimensions = ttk.Frame(card, style="Card.TFrame")
        dimensions.pack(fill="x", pady=(12, 8))

        ttk.Label(
            dimensions, text="Processes:", background="white"
        ).grid(row=0, column=0, sticky="w")
        ttk.Entry(
            dimensions, textvariable=self.process_var, width=8
        ).grid(row=0, column=1, padx=6)

        ttk.Label(
            dimensions, text="Resource types:", background="white"
        ).grid(row=1, column=0, sticky="w", pady=5)
        ttk.Entry(
            dimensions, textvariable=self.resource_var, width=8
        ).grid(row=1, column=1, padx=6)

        ttk.Label(
            card, text="Total vector (one row)", background="white"
        ).pack(anchor="w", pady=(6, 2))
        ttk.Entry(
            card, textvariable=self.total_var, width=36
        ).pack(fill="x")

        ttk.Label(
            card, text="Allocation matrix (one process per line)",
            background="white"
        ).pack(anchor="w", pady=(10, 2))
        self.allocation_text = tk.Text(
            card, width=36, height=7, font=("Consolas", 10),
            relief="solid", bd=1, wrap="none"
        )
        self.allocation_text.pack(fill="x")

        ttk.Label(
            card, text="Max matrix (one process per line)",
            background="white"
        ).pack(anchor="w", pady=(10, 2))
        self.max_text = tk.Text(
            card, width=36, height=7, font=("Consolas", 10),
            relief="solid", bd=1, wrap="none"
        )
        self.max_text.pack(fill="x")

        buttons = ttk.Frame(card, style="Card.TFrame")
        buttons.pack(fill="x", pady=(12, 4))
        ttk.Button(
            buttons, text="Load Sample", command=self.load_sample
        ).pack(side="left", padx=(0, 5))
        ttk.Button(
            buttons, text="Run Safety Check", style="Accent.TButton",
            command=self.run_safety
        ).pack(side="left")

        ttk.Separator(card).pack(fill="x", pady=12)

        # This section is part of the scrollable left card.
        ttk.Label(
            card, text="OPTIONAL: RESOURCE REQUEST",
            style="CardTitle.TLabel"
        ).pack(anchor="w")
        ttk.Label(
            card, text="Requesting process (0-based index)",
            background="white"
        ).pack(anchor="w", pady=(8, 2))
        ttk.Entry(
            card, textvariable=self.request_process_var, width=12
        ).pack(anchor="w")

        ttk.Label(
            card, text="Request vector (one value per resource)",
            background="white"
        ).pack(anchor="w", pady=(8, 2))
        ttk.Entry(
            card, textvariable=self.request_var, width=36
        ).pack(fill="x")

        ttk.Button(
            card, text="Check Request (Grant / Wait)",
            command=self.run_request
        ).pack(fill="x", pady=(10, 4))

        ttk.Label(
            card,
            text="Scroll down in this LEFT panel to see this section.",
            background="white", foreground="#65758b",
            wraplength=330
        ).pack(anchor="w", pady=(5, 2))

    def _update_left_scroll(self, _event=None):
        self.left_canvas.configure(
            scrollregion=self.left_canvas.bbox("all")
        )

    def _resize_left_content(self, event):
        self.left_canvas.itemconfigure(self.left_window, width=event.width)

    def _enable_mousewheel(self, widget):
        """Enable wheel scrolling only while pointer is over this widget."""
        widget.bind("<Enter>", lambda _event, w=widget: self._set_wheel_target(w))
        widget.bind("<Leave>", lambda _event: self._set_wheel_target(None))

    def _set_wheel_target(self, widget):
        self._wheel_target = widget

    def _on_mousewheel(self, event):
        target = getattr(self, "_wheel_target", None)
        if target is None:
            return

        # Windows uses delta in multiples of 120. Linux wheel buttons are
        # handled separately below.
        if event.delta:
            target.yview_scroll(int(-event.delta / 120), "units")
        return "break"

    def _on_mousewheel_linux_up(self, _event):
        target = getattr(self, "_wheel_target", None)
        if target is not None:
            target.yview_scroll(-1, "units")
        return "break"

    def _on_mousewheel_linux_down(self, _event):
        target = getattr(self, "_wheel_target", None)
        if target is not None:
            target.yview_scroll(1, "units")
        return "break"

    def load_sample(self):
        self.process_var.set("5")
        self.resource_var.set("3")
        self.total_var.set("10 5 7")
        allocation = (
            "0 1 0\n"
            "2 0 0\n"
            "3 0 2\n"
            "2 1 1\n"
            "0 0 2"
        )
        maximum = (
            "7 5 3\n"
            "3 2 2\n"
            "9 0 2\n"
            "2 2 2\n"
            "4 3 3"
        )
        self._set_text(self.allocation_text, allocation)
        self._set_text(self.max_text, maximum)
        self.request_process_var.set("1")
        self.request_var.set("1 0 2")
        self._write(
            "Sample data loaded. Click 'Run Safety Check' to simulate.\n",
            "heading", clear=True
        )
        self.left_canvas.yview_moveto(0)

    @staticmethod
    def _set_text(widget, value):
        widget.delete("1.0", "end")
        widget.insert("1.0", value)

    def _write(self, message, tag=None, clear=False):
        if clear:
            self.output.delete("1.0", "end")
        self.output.insert(
            "end", message + ("" if message.endswith("\n") else "\n"), tag
        )
        self.output.see("end")

    def _read_inputs(self):
        try:
            process_count = int(self.process_var.get())
            resource_count = int(self.resource_var.get())
        except ValueError:
            raise ValueError(
                "Process count and resource type count must be whole numbers."
            )

        if not (1 <= process_count <= 20):
            raise ValueError("Use 1–20 processes.")
        if not (1 <= resource_count <= 10):
            raise ValueError("Use 1–10 resource types.")

        total_parts = self.total_var.get().replace(",", " ").split()
        if len(total_parts) != resource_count:
            raise ValueError(
                f"Total vector must contain exactly {resource_count} values."
            )
        try:
            total = [int(value) for value in total_parts]
        except ValueError:
            raise ValueError("Total vector must contain whole numbers only.")

        if any(value < 0 for value in total):
            raise ValueError("Total vector cannot contain negative values.")

        allocation = parse_matrix(
            self.allocation_text.get("1.0", "end"),
            process_count, resource_count, "Allocation"
        )
        maximum = parse_matrix(
            self.max_text.get("1.0", "end"),
            process_count, resource_count, "Max"
        )
        validate_state(total, allocation, maximum)
        return total, allocation, maximum

    def run_safety(self):
        try:
            total, allocation, maximum = self._read_inputs()
            result = safety_algorithm(total, allocation, maximum)
        except ValueError as error:
            messagebox.showerror("Input Error", str(error))
            return

        self._write("", clear=True)
        self._write("BANKER'S SAFETY ALGORITHM", "heading")
        self._write("=" * 62)
        self._write(f"Total       : {total}")
        self._write(f"Allocation  : {allocation}")
        self._write(f"Max         : {maximum}")
        self._write(f"Need        : {result['need']}")
        self._write(f"Available   : {result['available']}")
        self._write("")
        self._write("STEP-BY-STEP TRACE", "heading")
        self._write("-" * 62)

        for line in result["trace"]:
            tag = (
                "good" if "SAFE STATE" in line or "Safe sequence" in line
                else "bad" if "UNSAFE STATE" in line
                else None
            )
            self._write(line, tag)

        self._write("")
        self._write(f"Final Finish flags: {result['finish']}")
        self._write(f"Final Work: {result['final_work']}")

    def run_request(self):
        try:
            total, allocation, maximum = self._read_inputs()
            process_index = int(self.request_process_var.get())
            request_parts = self.request_var.get().replace(",", " ").split()
            try:
                request = [int(value) for value in request_parts]
            except ValueError:
                raise ValueError("Request vector must contain whole numbers.")
            granted, trace = check_resource_request(
                total, allocation, maximum, process_index, request
            )
        except ValueError as error:
            messagebox.showerror("Request / Input Error", str(error))
            return

        self._write("", clear=True)
        self._write("RESOURCE-REQUEST CHECKING", "heading")
        self._write("=" * 62)
        for line in trace:
            tag = (
                "good" if "DECISION: GRANT" in line
                else "bad" if "DECISION: WAIT" in line or "UNSAFE STATE" in line
                else None
            )
            self._write(line, tag)

        self._write("")
        self._write(
            "Request result: " + ("GRANT" if granted else "WAIT"),
            "good" if granted else "bad"
        )


def main():
    root = tk.Tk()
    app = BankersGUI(root)

    # Global mouse-wheel handlers; the active panel is selected by pointer
    # enter/leave events. This supports Windows and Linux.
    root.bind_all("<MouseWheel>", app._on_mousewheel)
    root.bind_all("<Button-4>", app._on_mousewheel_linux_up)
    root.bind_all("<Button-5>", app._on_mousewheel_linux_down)

    root.mainloop()


if __name__ == "__main__":
    main()
