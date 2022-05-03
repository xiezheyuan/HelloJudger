# HelloJudger
A Simple Judger Written in Python.

指令：

```
python hellojudger.py 测试点文件夹 测试程序EXE [时限，单位为秒，默认为1]
```

当前 `hellojudger.py`：

```python
import sys
import os
import subprocess
import time
import prettytable
import colorama
import tqdm

TIME_OUT = 1

TLE = colorama.Back.YELLOW + colorama.Fore.WHITE + " Time Limit Exceed " + colorama.Back.RESET + colorama.Fore.RESET
RE = colorama.Back.MAGENTA + colorama.Fore.WHITE + " Runtime Error " + colorama.Fore.RESET + colorama.Back.RESET
WA = colorama.Back.RED + colorama.Fore.WHITE + " Wrong Answer " + colorama.Back.RESET + colorama.Fore.RESET
AC = colorama.Back.GREEN + colorama.Fore.WHITE + " Accepted " + colorama.Back.RESET + colorama.Fore.RESET
UN_AC = colorama.Back.RED + colorama.Fore.WHITE + " Unaccepted " + colorama.Back.RESET + colorama.Fore.RESET
UKE = "Unknown Error"


def hello_judger(args):
    global TIME_OUT
    args = args[1:]
    if len(args) < 2:
        print("Error:You give me too few argument.")
        os.system("pause")
        return
    if len(args) > 3:
        print("Error:You give me too many arguments.")
        os.system("pause")
        return
    test_folder = args[0]
    test_executable = args[1]
    if len(args) == 3:
        TIME_OUT = float(args[2])
    checks = {}
    ins = []
    outs = []
    for i in os.listdir(test_folder):
        if not os.path.isfile(os.path.join(test_folder, i)):
            continue
        if i.endswith(".in"):
            ins.append(i)
        elif i.endswith(".out"):
            outs.append(i)
    for i in ins:
        if i.replace(".in", ".out") in outs:
            checks[os.path.join(test_folder, i)] = os.path.join(test_folder, i.replace(".in", ".out"))
    per_testdata = 100.0 / len(checks)
    total = 0
    all_consuming = 0.0
    table = prettytable.PrettyTable(["Test Data", "Status", "Point", "Consuming"])
    for in_ in tqdm.tqdm(checks, desc="Judging"):
        now = []
        out_ = checks[in_]
        now.append(str("{} / {}".format(in_.split('\\')[-1], out_.split('\\')[-1])))
        in_stream = open(in_, "r")
        out_stream = open(out_, "r")
        pa_stream = open(".checking.out", "w")
        start = time.time()
        result = subprocess.Popen([test_executable], stdin=in_stream, stdout=pa_stream)
        timeout = False
        while result.poll() != 0:
            end = time.time()
            if end - start > TIME_OUT:
                timeout = True
                break
        result.terminate()
        pa_stream.close()
        pa_stream = open(".checking.out", "r")
        ans_result = out_stream.read()
        ans_list = ans_result.splitlines()
        ans_result = ""
        for i in ans_list:
            ans_result += i.rstrip(" ")
        out_result = pa_stream.read()
        pa_list = out_result.splitlines()
        out_result = ""
        for i in pa_list:
            out_result += i.rstrip(" ")
        if timeout:
            now.append(TLE)
            now.append("0 / {:.2f}".format(per_testdata))
        elif result.returncode != 0:
            now.append(RE)
            now.append("0 / {:.2f}".format(per_testdata))
        elif ans_result != out_result:
            now.append(WA)
            now.append("0 / {:.2f}".format(per_testdata))
        else:
            now.append(AC)
            now.append("{:.2f} / {:.2f}".format(per_testdata, per_testdata))
            total += per_testdata
        now.append("{:.3f} s".format(end - start))
        all_consuming += (end - start)
        table.add_row(now)
        in_stream.close()
        out_stream.close()
        pa_stream.close()
        # noinspection PyBroadException
        try:
            os.remove(".checking.out")
        except BaseException:
            pass
    total_t = ["Total"]
    if round(total) == 100:
        total_t.append(AC)
    else:
        total_t.append(UN_AC)
    total_t.append("{:.2f} / 100".format(total))
    total_t.append("{:.3f} s".format(all_consuming))
    table.add_row(total_t)
    print(table)


if __name__ == "__main__":
    # noinspection PyBroadException
    try:
        hello_judger(sys.argv)
    except Exception as err:
        print(UKE)
        print(repr(err))
    os.system("pause")

```
