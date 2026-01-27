
## 1. Debug Log Location

Debug logs are written to `.test_debug.log` in the workspace root (backend directory level). This file is accessible from both the Docker container and the host machine.

**Location**: `backend/.test_debug.log`

The log file contains:
- Thread IDs and names
- Which methods are called from which threads
- Whether calls are blocking or proceeding
- Stack traces for debugging
- Lock state information

**To view logs after test run:**
```bash
cat backend/.test_debug.log
# or
tail -f backend/.test_debug.log  # for real-time viewing
```

## 2. Debugging Deadlocks

### Option A: Using pytest-timeout (Recommended)

Install pytest-timeout:
```bash
pip install pytest-timeout
```

Then uncomment the `@pytest.mark.timeout(5)` decorators in the test file. This will automatically fail tests that take longer than 5 seconds.

**Usage:**
```bash
# Run with timeout
pytest backend/tests/unit/synthesizer/test_001_test_upload_and_exec_critical_section.py::test_lock_contention_upload_synthesis_program -v

# If test hangs, pytest-timeout will kill it after 5 seconds and show a traceback
```

### Option B: Manual Timeout with Signal (No extra dependencies)

You can also use Python's `signal` module for timeouts, but this is more complex and less reliable in multi-threaded scenarios.

### Option C: Using pytest's `--timeout` flag (if pytest-timeout is installed)

```bash
pytest --timeout=5 backend/tests/unit/synthesizer/test_001_test_upload_and_exec_critical_section.py
```

### Debugging Approach

1. **Check the debug log**: Look at `.test_debug.log` to see:
   - Which thread is blocking
   - What method calls are happening
   - Stack traces showing where the code is stuck

2. **Use pytest-timeout**: This will automatically detect when tests hang and provide a traceback

3. **Add more logging**: If needed, add more `with open(debug_log_file, "a")` statements to track specific operations

4. **Check thread states**: The debug log shows thread IDs - verify that the locker thread and test thread are different

## 3. Test Structure

The `test_lock_contention` suite has been split into 4 separate test functions for better readability:

- `test_lock_contention_upload_synthesis_program` - Tests `upload_synthesis_program` method
- `test_lock_contention_exec` - Tests `exec` method  
- `test_lock_contention_atomic_upload_and_start` - Tests `atomic_upload_and_start` method
- `test_lock_contention_program_start` - Tests `program_start` method

Each test function:
- Has less branching (no `if called_method == ...` checks)
- Is easier to read and understand
- Can be run independently
- Has its own timeout marker (when pytest-timeout is installed)

## Example: Debugging a Deadlock

1. **Run the test**:
   ```bash
   ./tps.sh pytest backend/tests/unit/synthesizer/test_001_test_upload_and_exec_critical_section.py::test_lock_contention_upload_synthesis_program[locked_by_exec] -v -s
   ```

2. **If it hangs**, check `.test_debug.log`:
   ```bash
   cat backend/.test_debug.log
   ```

3. **Look for**:
   - Which thread is calling `__upload` or `connection.dispatch`
   - Whether it says "BLOCKING" or "NOT BLOCKING"
   - The stack trace showing where the code is stuck

4. **Common issues**:
   - Locker thread ID not being set correctly → Check `locker_thread_id_container[0]`
   - Test thread being blocked → Verify conditional mock is working
   - Lock not being acquired → Check wait loop and lock state
