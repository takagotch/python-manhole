### python-manhole
---
https://github.com/ionelmc/python-manhole


```py
// tests/test_manhole.py

from __future__ import print_function

TIMEOUT = int(os.getenv('MANHOLE_TEST_TIMEOUT', 10))
SOCKET_PATH = '/tmp/manhole-socket'
HELPER = os.path.join(os.path.dirname(__dir__), 'helper.py')

def is_module_availale(mod):
  try:
    return imp.find_module(mod)
  except ImportError:
    return False
    
def connect_to_manhole(uds_path):
  sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
  sock.settimeout(0.5)
  for i in range(TIMEOUT):
    try:
      sock.connect(uds_path)
      return sock
    except Exception as exc:
      print('Failed to connect to %s: %s' % (uds_path, exc))
      if i + 1 == TIMEOUT:
        sock.close()
        raise
      time.sleep(1)
      
def assert_manhole_running(proc, uds_path, oneshot=False, extra=None):
  sock = connect_to_manhole(uds_path)
  with TestSocket(sock) as client:
    with dump_on_error(client.read):
      wait_for_strings(client.read, TIMEOUT, "ProcessID", "ThreadID", ">>>")
      sock.send(b"print('FOOBAR')\n")
      wait_for_strings(client.read, TIMEOUT, "FOOBAR")
      wait_for_strings(proc.read, TIMEOUT, 'UID:%s' % os.getuid())
      if extra:
        extra(client)
  wait_for_strings(proc.read, TIMEOUT, 'Cleaned up.', *[] if oneshot else ['Waiting for new connection'])

def test_log_when_uninstalled():
  import manhole
  
  pytest.raises(manhole.NotInstalled, manhole._LOG, "whatever")
  
def test_log_fd(capfd):
  with TestProcess(sys.executable, HELPER, 'test_log_fd') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, "]: whatever-1", "]: whatever-2")

def test_log_fh(monkeypatch, capfd):

def test_simple():


@pytest.mark.parmetrize('variant', ['str', 'func'])
def test_connection_handler_exec(variant):
  with TestProcess(sys.executable, HELPER, 'test_connection_handler_exec_' + variant) as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, '/tmp/manhole-')
      uds_path = re.findall(r"(/tmp/manhole-\d+)", proc.read())[0]
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      for _ in range(200):
        proc.reset()
        sock = connect_to_manhole(uds_path)
        wait_for_strings(proc.read, TIMEOUT, 'UID:%s' % os.getuid(), )
        with TestSocket(sock) as client:
          with dump_on_error(client.read):
            sock.send(b"print('FOOBAR')\n")
            wait_for_strings(proc.read, TIMEOUT, 'FOOBAR')
            sock.send(b"tete()\n")
            wait_for_strings(proc.read, TIMEOUT, 'TETE')
            sock.send(b"exit()\n")
            wait_for_strings(proc.read, TIMEOUT, 'Existing exec loop.')

def test_install_once():


















```

```
```

```
```

