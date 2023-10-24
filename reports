# 第三章学习笔记

## 编程题

1. 修改数据结构

    要想实现task_info系统调用，首先考虑在TCB中增加相应的信息：

    ```rust
    pub struct TaskControlBlock {
        /// The task status in it's lifecycle
        pub task_status: TaskStatus,
        /// The task context
        pub task_cx: TaskContext,
        /// The syscall times
        syscall_times: [u32; MAX_SYSCALL_NUM],
        /// The time process start to run
        start_time: usize
    }
    ```

    注意TaskControlBlock中的成员变量改变了，也需要修改TaskControlBlock的初始化方式。为了方便旧的结构体创建方式，提供了new方法：

    ```rust
    impl TaskControlBlock {
        pub fn new(task_status: TaskStatus, task_cx: TaskContext) -> Self {
            TaskControlBlock {
                task_status,
                task_cx,
                syscall_times: [0u32; MAX_SYSCALL_NUM],
                start_time: 0
            }
        }
    }
    ```

    并修改TASK_MANAGER中创建TCB的方式：

    ```rust
    lazy_static! {
        /// Global variable: TASK_MANAGER
        pub static ref TASK_MANAGER: TaskManager = {
            let num_app = get_num_app();
            // Apply new method to create TaskControlBlock
            let mut tasks = [TaskControlBlock::new(
                TaskStatus::UnInit,
                TaskContext::zero_init(),
            ); MAX_APP_NUM];
            for (i, task) in tasks.iter_mut().enumerate() {
                task.task_cx = TaskContext::goto_restore(init_app_cx(i));
                task.task_status = TaskStatus::Ready;
            }
            TaskManager {
                num_app,
                inner: unsafe {
                    UPSafeCell::new(TaskManagerInner {
                        tasks,
                        current_task: 0,
                    })
                },
            }
        };
    }
    ```

2. 更新系统调用次数信息

    找到系统调用在trap_handler中的处理逻辑，增加一个用于更新当前task系统调用次数的方法update_current_syscall_count：

    ```rust
    match scause.cause() {
        Trap::Exception(Exception::UserEnvCall) => {
            // jump to next instruction anyway
            cx.sepc += 4;
            // update TCB system call count
            update_current_syscall_count(cx.x[17]);
            // get system call return value
            cx.x[10] = syscall(cx.x[17], [cx.x[10], cx.x[11], cx.x[12]]) as usize;
        }
    ......
    ```

    然后在task模块以及TaskManager中增加相应的方法：

    ```rust
    // task/mod.rs
    pub fn update_current_syscall_count(syscall_id:usize) {
        TASK_MANAGER.update_syscall_count(syscall_id);
    }
    ```

    ```rust
    // task/mod.rs
    pub fn update_current_syscall_count(syscall_id:usize) {
        TASK_MANAGER.update_syscall_count(syscall_id);
    }
    ```

    ```rust
    // impl TaskManager
    fn update_syscall_count(&self, syscall_id:usize) {
        let mut inner = self.inner.exclusive_access();
        let current = inner.current_task;
        inner.tasks[current].syscall_times[syscall_id] += 1;
    }
    ```

3. 设置进程第一次被调度的时刻

   在内核的调度函数run_next_task中加入简单的判断逻辑：

   ```rust
    fn run_next_task(&self) {
        if let Some(next) = self.find_next_task() {
            let mut inner = self.inner.exclusive_access();
            let current = inner.current_task;
            inner.tasks[next].task_status = TaskStatus::Running;

            // If next process is sheduled the first time, initiate start_time
            if inner.tasks[next].start_time == 0 {
                inner.tasks[next].start_time = timer::get_time_us();
            }
    ......
   ```

4. 实现sys_task_info系统调用

    实现sys_task_info：

    ```rust
    pub fn sys_task_info(_ti: *mut TaskInfo) -> isize {
        trace!("kernel: sys_task_info");
        if let Some(task_info) = get_current_task_info() {
            unsafe {
                *_ti = task_info;
            }
            0
        }
        else {
            -1
        }
    }
    ```

    在TaskManage中实现主体逻辑：

    ```rust
    fn get_task_info(&self, task_id:Option<usize>) -> Option<TaskInfo> {
        let inner = self.inner.exclusive_access();
        // 
        let task_id = if let Some(id) = task_id {id} else {inner.current_task};
        if task_id < inner.tasks.len() {
            Option::Some(TaskInfo {
                status: inner.tasks[task_id].task_status,
                syscall_times: inner.tasks[task_id].syscall_times.clone(),
                time: match inner.tasks[task_id].start_time {
                    0 => 0,
                    // transfer to millisecond
                    t => (timer::get_time_us() - t) / 1000,
                },
            })
        }
        else {
            Option::None
        }
    }
    ```
