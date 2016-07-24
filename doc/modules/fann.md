---
layout: doc_modules
title: Neural network module
---
# Neural network module

Neural network module is an experimental module that allows to perform post-classification of messages based on their current symbols and some training corpus obtained from the previous learns.

To use this module, you need to build rspamd with `libfann` support. It is normally enabled if you use pre-built packages, however, it could be specified using `-DENABLE_FANN=ON` to `cmake` command during build process.

The idea behind this module is to learn which symbols combinations are common for spamd and which are common for ham. To achieve this goal, fann module studies log files via `log_helper` worker unless gathering some reasonable amount of log samples (`1k` by default). Neural network is learned for spam when a message has `reject` action (definite spam) and it is learned as ham when a message has negative score. You could also use your own criteria for learning.

Training is performed in background and after some amount of trains (`1k` again) neural network is updated on the disk allowing scanners to load and update their own data.

After some amount of such iterations (`100` by default), the training process removes old neural network and starts training new one. This is done to ensure that old data does not influence on the current processing. The neural network is also reset when you add or remove rules from rspamd. Once trained, neural network data is saved into file so it could persist between restarts. The current training epoch is however vanished upon restart.

## Configuration

First of all, you need a special worker called `log_helper` to accept rspamd scan results. This logger has a trivial setup:

~~~ucl
worker "log_helper" {
  count = 1;
}
~~~

Then you'd need to setup fann plugin:

~~~ucl
fann_scores {
  fann_file = "${DBDIR}/data.fann"; # Used to store ANN file on disk
  train {
    max_train = 10k; # Number of trains per epoch
    max_epoch = 1k # Number of epoch while ANN data is valid
    spam_score = 8; # Score to learn spam
    ham_score = -2; # Score to learn ham
  }
  use_settings = false; # If enabled, then settings-id could switch this module to another FANN
}
~~~

## Settings usage

TODO
