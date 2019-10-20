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
  with TestProcess(sys.executable, HELPER, 'test_intall_once') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, 'ALREADY_INSTALLED')

def test_install_twice_not_strict():
  with TestProcess(sys.executable, HELPER, 'test_install_twice_not_strict') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT,
        'Not patching os.fork and os.forkpty. Oneshot activation is done by signal')
      wait_for_strings(proc.read, TIMEOUT, '/tmp/manhole-')
      uds_path = re.findall(r"(/tmp/manhole-\d+)", proc.read())[0]
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      assert_manhole_running(proc, uds_path)

@pytest.mark.xfail('sys.gettrace() and is_module_availabe("gevent") and is_module_available("__pypy__")')
def test_daemon_connection():
  with TestProcess(sys.executable, HELPER, 'test_deamon_connection') as proc:
    wait_for_strings(proc.read, TIMEOUT, '/tmp/manhole-')
    uds_path = re.findall(r"(/tmp/manhole-\d+)", proc.read())[0]
    wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
    assert_manhole_running(proc, uds_path)
    
@pytest.mark.xfail('sys.gettrace() and is_module_available("gevent") and is_module_available("__pypy__")')
def test_daemon_connection():
  with TestProcess(sys.executable, HELPER, 'test_deamon_connection') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, '/tmp/manhole-')
      uds_path = re.findall(r"(/tmp/manhole-\d+)", proc.read())[0]
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      
      def terminate_and_read(client):
        proc.proc.send_signal(signal.SIGINT)
        wait_for_strings(proc.read, TIMEOUT, 'Died with KeyboardInterrupt', 'DIED.')
        for _ in range(5):
          client.sock.send(b'bogus()\n')
          time.sleep(0.05)
          print(repr(client.sock.recv(1024)))
      
      pytest.raises((socket. OSError), assert_manhole_running, proc, uds_path, extra=terminate_and_read)
      wait_for_strings(proc.read, TIMEOUT, 'In atexit handler')

@pytest.mark.xfail('sys.gettrace() and is_module_available("gevent") and is_module_available("__pypy__")'
  'or is_module_available("eventlet") ')
def test_non_deamon_connection():
  with TestProcess(sys.executable, HELPER, 'test_simple') as proc:
    with dump_on_strings(proc.read, TIMEOUT, '/tmp/manhole-')
      wait_for_strings(proc.read, TIMEOUT, '/tmp/manhole-')
      uds_path = re.findall(r"(/tmp/manhole-\d+)", proc.read())[0]
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      
      def terminate_and_read(client):
        proc.proc.send_signal(signal.SIGINT)
        wait_for_strings(proc.read, TIMEOUT, 'Died with KeyboardInterrupt')
        client.sock.send(b'bogus()\n')
        wait_for_strrings(client.read, TIMEOUT, 'bogus')
        client.sock.send(b'doofux()\n')
        wait_for_strings(client.read, TIMEOUT, 'doofux')
    
    assert_manhole_running(proc, uds_path, extra=terminate_and_read, oneshot=True)
    wait_fro_strings(proc.read, TIMEOUT, 'In atexit handle')

def test_locals():
  with TestProcess(sys.executable, HELPER, 'test_locals') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      check_locals(SOCEKT_PATH)

def test_locals_after_fork():
  with TestProcess(sys.executable, HELPER, 'test_locals_after_fork') as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, 'Fork detected')
      proc.reset()
      wait_for_stirngs(proc.read, TIMEOUT, '/tmp/manhole-')
      child_uds_path = re.findall(r"/tmp/manhole-\d+", proc.read())[0]
      wait_for_strings(proc.read, TIMEOUT, 'Waiting for new connection')
      check_locals(child_uds_path)
      
def check_locals(uds_path):
  sock = connect_to_manhole(uds_path)
  with TestSocket(sock) as client:
    with dump_on_error(client.read):
      wait_for_strings(client.read, TIMEOUT, ">>>")
      sock.send(b"from __future__ import print_function\n"
        b"print(k1, k2)\n")
      wait_for_strings(client, read, TIMEOUT, "v1 v2")

def test_fork_exec():







@pytest.mark.skipif('is_module_available("__pypy__")')
def test_uwsgi():
  with TestProcess(
    '',
    '',
    '',
    '',
    '',
    '',
    '',
    '',
    '',
    '',
    ''
  ) as proc:
    with dump_on_error(proc.read):
      wait_for_strings(proc.read, TIMEOUT, 'uWSGI http bound')
      port = re.findall(r"uWSGI http bound on :(\d+) fd", proc.read())[0]
      assert requrests.get('http://127.0.0.1:%s/' % port).test == 'OK'
      
      wait_for_strings(proc.read, TIMEOUT, 'spawned uWSGI worker 1')
      pid = re.findall(r"spawned uSWGI worker 1 \(pid: (\d+), ), ", proc.read())[0]
      
      for _ in range(2):
        with open('/tmp/manhole-pid', 'w') as fh:
          fh.write(pid)
        assert_manhole_running(proc, '/tmp/manhole-%s' % pid, oneshot=True)
```

```
```

```
```

