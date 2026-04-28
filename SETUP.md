# setup notes (group 20)

these are the steps that worked for me getting zhe's repo running.
saving them so the rest of the group doesn't have to figure it out from scratch.

## what i used
- ubuntu 24.04 on wsl2 (windows 11)
- python 3.12.3
- node 22
- isabelle 2025-2
- isabelle-client 1.1.0 (whatever pip pulled in late april 2026)

## if you're on windows, use wsl
i tried doing this on native windows first and it was painful.
the `isabelle` command on windows is just the gui launcher and doesn't
actually take cli args. switched to wsl and it just worked.
install with `wsl --install` from an admin powershell, then reboot.

mac should be fine to just follow the steps directly.

## steps

1. isabelle 2025-2
   - linux: https://isabelle.in.tum.de/website-Isabelle2025-2/dist/Isabelle2025-2_linux.tar.gz
   - mac: get the .dmg from the same page
   - add the bin folder to your path. test with `isabelle version`.

2. python venv

       python3 -m venv .venv
       source .venv/bin/activate
       pip install -U pip
       pip install -r requirements.txt
       pip install torch --index-url https://download.pytorch.org/whl/cpu
       pip install -U sentence-transformers

3. node + gemini cli
   need node 22, ubuntu's default apt gives you 18 which is too old
   and the gemini cli crashes. do this instead:

       # linux
       curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
       sudo apt install -y nodejs
       # mac
       brew install node

   then:

       sudo npm install -g @google/gemini-cli

4. gemini api key
   get one at https://aistudio.google.com/app/apikey (free).
   each person makes their own, don't share keys.
   add to your ~/.bashrc or ~/.zshrc:

       export GEMINI_API_KEY=your_key_here
       export GEMINI_CLI_TRUST_WORKSPACE=true

   the trust thing is needed otherwise gemini cli refuses to run in
   a directory you haven't clicked through interactively.

5. test it works:

       gemini -m gemini-2.5-flash -p "hi"
       LLM_DEBUG=1 python -m prover.cli --model "gemini:gemini-2.5-flash" --sledge --beam 3 --max-depth 5 --timeout 120 --trace

   should end with "SUCCESS | depth: 1" and a one-line proof.

## stuff i had to patch
the latest isabelle-client (1.1.0) changed how some responses come back
and zhe's code didn't handle it. two fixes already in this repo:

- prover/cli.py: session_start now returns a list instead of a string,
  pull session_id out of the SessionStartRegularResponse
- prover/isabelle_api.py: response bodies are pydantic objects now,
  added a model_dump() check at the top of _decode_body_to_dict

without these every proof attempt silently fails because finished_ok
can't read the response.
