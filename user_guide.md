# User Guide
## Configuration File
Under the `nntrader/nntrader` directory, there is a json file called `net_config.json`,
 holding all the configuration of the agent and could be modified outside the program code.
### Network Topology
* `"layers"`
    * layers list of the CNN, including the output layer
    * `"type"`
        * domain is {"ConvLayer", "FullyLayer", "DropOut", "MaxPooling",
        "AveragePooling", "LocalResponseNormalization", "SingleMachineOutput",
        "LSTMSingleMachine", "RNNSingleMachine"}
    * `"filter shape"`
        * shape of the filter (kernal) of the Convolution Layer
* `"input"`
    * `"window_size"`
        * number of columns of the input matrix
    * `"coin_number"`
        * number of rows of the input matrix
    * `"feature_number"`
        * number of features (just like RGB in computer vision)
        * domain is {1, 2, 3}
        * 1 means the feature is ["close"], last price of each period
        * 2 means the feature is ["close", "volume"]
        * 3 means the features are ["close", "high", "low"]

### Market Data
* `"input "`
    * `"start_date"`
        * start date of the global data matrix
        * format is yyyy/MM/dd
    * `"end_date"`
        * start date of the global data matrix
        * format is yyyy/MM/dd
    * `"volume_average_days"`
        * number of days of volume used to select the coins
    * `"online"`
        * if it is not online, the program will select coins and generate inputs
        from the local database.
        * if it is online, new data that dose not exist in the database would be saved

## Training and Tuning the hyper-parameters
1. First, change the content in the `nntrader/nntrader/net_config.json` file.
2. make sure current directory is under `nntrader` and type `python main.py --mode=generate --repeat=1`
    * this will make 1 subfolders under the `train_package`
    * in each subfloder, there is a copy of the `net_config.json`
    * `--repeat=n`, n could follow any positive integers. The random seed of each the subfolder ranges from 0 to n-1.
3. type `python main.py --mode=train --processes=1`
    * this will start training one by one of the n floders created just now
    * do not start more than 1 processes if you want to download data online
4. after that, check the summary of the training in `nntrader/train_package/train_summary`
5. tune the hyper-parameters based on the summary, and goto 1 again.

## Logging
There are three types of logging of each training.
* In each subfloder
    * There is a text file called `programlog`, which is the log generated by the running programming.
    * There is a `tensorboard` folder saves the data about the training process which could be viewed by tensorboard.
        * type `tensorboard --logdir=train_package/1` to use tensorboard
* The summary infomation of this training, including network configuration, portfolio value on validation set and test set etc., will be saved in the `train_summary.csv` under `train_pakage` folder

## Save and Restore of the Model
* The trained weights of the network are saved at `train_package/1` named as `netfile` (including 3 files). 

## Download Data
* Type `python main.py --mode=download_data` you can download data without starting training
* The program will use the configurations in `nntrader/nntrader/net_config` to select coins and
  download necessary data to train the network.
* The downloading speed could be very slow and sometimes even have error in China. See #4 for details.

## Back-test
* Type `python main.py --mode=backtest --algo=1` to conduct
back test with rolling train(i.e. online learning in supervised learning)
on the target model.
paper trading
* `--algo` could be either the name of traditional method or the index of training folder

## Tradition Agent
see progress on [wiki](https://github.com/ZhengyaoJiang/nntrader/wiki/Current-Progress-of-Traditional-Agents)

OLPS summary:

![](https://github.com/DexHunter/nntrader/blob/dev/images/olps_algo.png)

## Ploting
* type `python main.py --mode=plot --algos=crp,olmar,40 --labels=crp,olmar,nnagent
`,for example, to plot
* `--algos` could be the name of the tdagent algorithms or
the index of nnagent
* `--labels` is the name that will be shown in the legend
* result is
![](http://oan6f7zbh.bkt.clouddn.com/17-4-23/91567996-file_1492914862957_4f40.png)
* a sample csv file https://drive.google.com/open?id=0Bz5He3MSOBUtNnZFRGFaRUhDZW8

## present backtest results in a table
* type `python main.py --mode=table --algos=40,olmar,ons --labels=nntrader,olmar,ons`
* `--algos` and `--lables` are the same as in plotting case
* result is like:
```
          max_drawdown  portfolio_value  sharpe_ratio
nntrader      0.360347       156.357158      0.099530
olmar         0.452494         7.968085      0.045061
ons           0.231059         1.799372      0.033449
```
* use `--format` arguments to change the format of the table,
 could be `raw` `html` or `latex`
    * sample of latex format:

```
\begin{tabular}{lrrr}
\toprule
{} &  max\_drawdown &  portfolio\_value &  sharpe\_ratio \\
\midrule
nntrader &      0.360347 &       156.357158 &      0.099530 \\
olmar    &      0.452494 &         7.968085 &      0.045061 \\
ons      &      0.231059 &         1.799372 &      0.033449 \\
\bottomrule
\end{tabular}

```
