# pb
最后的pb文件,但是没有优化模型

In this tutorial i use the the tensorflow android calssifier, if you want to build it, it will take you some time, as you’ll need to install the NDK, Bazel, and the total build time with Android Studio will take around 40 minutes, of course you can build it by yourself by this url (https://github.com/tensorflow/tensorflow). But for someone who just want to try this demo, i pushed my build files in my github.

1 step:  Retraining your own model, and you can use docker for traning. (retraing your own tensorflow model tutorial:https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/#0)

2 step:  After retraining, you will get two files like: "retrained_graph.pb", "retrained_labels.txt"

3 step:  We have our model and label. However, if we try to import it in our Android sample, after build your project, we will get an error:  
Op BatchNormWithGlobalNormalization is not available in GraphDef version 21. It has been removed in version 9. Use tf.nn.batch_normalization().

Because you did not optimize the model, so first we need need to optimize by using a tool named "optimize_for_inference" in your tensorflow folder, run two codes follow:(you have to install bazel which is a build tool coordinates builds and run tests and you can install this tool: https://bazel.build/versions/master/docs/install.html)
         
         ./configure (just select all default values)
         bazel build tensorflow/python/tools:optimize_for_inference 
Building the tool should take around 1 hour in my google cloud VM, so to save your time, you can commit your current docker container by follow code:
         
         exit
         docker ps -a
         docker commit container ID new_name
And then start your new container with:
         
         docker run -it -v $HOME/gaobinlove:/tensorflow <new_name>
         cd tensorflow
         
4 step:  When you finish building tool, you can run follow code to optimeze your model:
         
         bazel-bin/tensorflow/python/tools/optimize_for_inference \
            --input=/pb/retrained_graph.pb \
            --output=/pb/retrained_graph_optimized.pb \
            --input_names=Mul \
            --output_names=final_result
in this step it just take about 2 min, and this script will generate a "retrained_graph_optimized.pb" file you will now be able into import in your Android project.

Step optional:  When you get your own model, you can see your model's size is 87 Mb, its not a problem, but you can run codes follow to reduce the size of the model:
         
         bazel build tensorflow/contrib/quantization/tools:quantize_graph
         bazel-bin/tensorflow/contrib/quantization/tools/quantize_graph \
           –input=/tf_files/optimized_graph.pb \
           –output=/tf_files/rounded_graph.pb \
           –output_node_names=final_result \
           –mode=weights_rounded
           
5 step:  In the end, you can now delete the previous ImageNet model from our Android app’s assets folder and place the new model (retrained_graph_optimized.pb and retrained_labels.txt) instead.


