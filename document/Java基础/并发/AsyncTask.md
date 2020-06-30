多线程的应用在Android开发中是非常常见的，常用方法主要有：
1、继承Thread类
2、实现Runnable接口
3、Handler
4、AsyncTask
5、HandlerThread

AsyncTask
定义：
1、封装好的轻量级异步类，该类属于抽象类，使用时需实现子类
 public abstract class AsyncTask<Params, Progress, Result> { 
 ... 
 }


作用：
1、实现多线程：在工作线程中执行任务，如：耗时任务；
2、异步通信、消息传递：实现工作线程&主线程之间的通信，即：将工作线程的执行结果传递给主线程，从而在主线程中执行相关的UI操作；
注：保证线程安全

优点：
1、方便实现异步通信：不需使用“ 任务线程（如：继承Thread类）+ Handler ” 的复杂组合；
2、节省资源：采用线程池中的缓存线程 + 复用线程，避免了频繁创建&销毁线程带来的系统资源开销；

类&方法：
1、类定义：AsyncTask 类属于抽象类，即使用时需实现子类；
 public abstract class AsyncTask<Params, Progress, Result> { 
 ... 
 }
// 类中参数为3种泛型类型
// 整体作用：控制AsyncTask子类执行线程任务时各个阶段的返回类型
// 具体说明：
    // a. Params：开始异步任务执行时传入的参数类型，对应excute（）中传递的参数
    // b. Progress：异步任务执行过程中，返回下载进度值的类型
    // c. Result：异步任务执行完成后，返回的结果类型，与doInBackground()的返回值类型保持一致
// 注：
    // a. 使用时并不是所有类型都被使用
    // b. 若无被使用，可用java.lang.Void类型代替
    // c. 若有不同业务，需额外再写1个AsyncTask的子类
}


2、核心方法：
1）常用方法：

2）方法执行顺序：


使用步骤：
1、创建 AsyncTask 子类 & 根据需求实现核心方法
2、创建 AsyncTask子类的实例对象（即 任务实例）
3、手动调用execute(（）从而执行异步线程任务
/** 自定义任务 */
class MyTask extends AsyncTask<String, Integer, String> {

    // String：开始异步任务执行时传入的参数类型，对应excute（）中传递的参数
    // Integer：异步任务执行过程中，返回下载进度值的类型
    // ICallback：异步任务执行完成后，返回的结果类型，与doInBackground()的返回值类型保持一致

    /** 执行前初始化 */
    @Override
    protected void onPreExecute() {
        textView.setText("开始加载...");
    }

    /** 执行并返回结果 */
    @Override
    protected String doInBackground(String... strings) {
        try {
            int count = 0;
            int length = 1;
            while (count<99) {
                count += length;
                // 可调用publishProgress（）显示进度, 之后将执行onProgressUpdate（）
                publishProgress(count);
                // 模拟耗时任务
                Thread.sleep(50);
            }
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
        return null;
    }

    /** 当前执行进度 */
    @Override
    protected void onProgressUpdate(Integer... progresses) {
        progressBar.setProgress(progresses[0]);
        textView.setText("加载中..." + progresses[0] + "%");
    }

    /** 执行完成 */
    @Override
    protected void onPostExecute(String s) {
        textView.setText("加载完成！！！");
        progressBar.setProgress(100);

    }

    @Override
    protected void onCancelled(String s) {
        textView.setText("已取消");
        progressBar.setProgress(0);
    }

}


4、根据不同的api采用并行模式进行开启
 // 根据不同的api采用并行模式进行开启
 if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {      
     new MyAsycTask().executeOnExecutor(THREAD_POOL_EXECUTOR,"参数");
} else {
    new MyAsycTask().execute("参数");
}


注意点：
1、生命周期：
不与任何组件绑定生命周期，在Activity或Fragment中使用时应在onDestory()方法中调用cancel(boolean)进行销毁；
2、内存泄漏：
若AsyncTask被声明为Activity的非静态内部类，当Activity需销毁时，会因AsyncTask保留对Activity的引用 而导致Activity无法被回收，最终引起内存泄露，AsyncTask应被声明为Activity的静态内部类；
3、线程任务执行结果丢失：
当Activity重新创建时（屏幕旋转 / Activity被意外销毁时后恢复），之前运行的AsyncTask（非静态的内部类）持有的之前Activity引用已无效，故复写的onPostExecute()将不生效，即无法更新UI操作，在Activity恢复时的对应方法 重启 任务线程；
4、cancel(boolean mayInterruptIfRunning)方法调用不当会失效：
如果的doInBackground()中有不可打断的方法则cancel方法可能会失效，比如BitmapFactory.decodeStream()， IO等操作，也就是即使调用了cancel方法，onPostExecute(Result result)方法仍然有可能被回调。
5、Activity被回收，操作UI时出现空指针异常：
如果在Activity中使用静态AsynTask子类，依然无法高枕无忧，虽然静态AsyncTask子类中不会持有Activity的隐式引用。但是AsyncTask的生命周期依然可能比Activity的长，当Activity进行销毁，AsyncTask执行回调onPostExecute(Result result)方法时，如果在onPostExecute(Result result)中进行了UI操作，UI对象会随着Activity的回收而被回收，导致UI操作报空指针异常。
6、多个异步任务同时开启时，存在串行并行问题：
当多个异步任务调用方法execute()同时开启时，api的不同会导致串行或并行的执行方式的不同。
在Android 1.6(Donut)之前，多个异步任务串行执行
从Android 1.6到Android 2.3(Gingerbread)，多个异步任务并行执行
Android 3.0（Honeycomb）到现在，如果调用方法execute()开启是串行，调用executeOnExecutor(Executor)是并行



相关问答：
1、为什么AsyncTask在主线程创建执行？
因为AsyncTask需要在主线程创建InternalHandler，以便onProgressUpdate， onPostExecute ， onCancelled 可以正常更新UI。
2、为什么AsyncTask不适合特别耗时任务？
AsyncTask实际上是一个线程池。如果有线程长时间占用，且没有空闲，则其他线程只能处于等待状态，会造成阻塞。
3、AsyncTask内存泄漏问题？
如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对创建了AsyncTask的Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄露。
4、AsyncTask执行问题？
每个任务只能执行一次，再次执行时会返回异常。
