## utility

- **无状态 (Stateless) 
    这就像一个普通的计算器。你输入 $3 \times 5$，它输出 $15$，然后立马把结果忘掉。它不需要在肚子里专门留一块内存去“记住”你昨天算了什么（不需要 `self`）。

- **低耦合 (Low Coupling)
    那个画图函数 `plot_learning_curve` 根本不在乎你算的是新冠预测、股票走势，还是你们专业的抗干扰天线信号。只要你给它两列数字，它就照画不误。**这就叫解耦**。
- **高复用 (Reusable)
    虽然它在一次执行中是用完即焚的，但正因为它是独立解耦的，你以后如果写新的论文、做新的项目，**完全可以直接把这三个 `def` 复制粘贴到新代码里，一行都不用改就能直接用！**

```python


def get_device():

''' Get device (if GPU is available, use GPU) '''

return 'cuda' if torch.cuda.is_available() else 'cpu'

  

def plot_learning_curve(loss_record, title=''):

''' Plot learning curve of your DNN (train & dev loss) '''

total_steps = len(loss_record['train'])

x_1 = range(total_steps)

x_2 = x_1[::len(loss_record['train']) // len(loss_record['dev'])]

figure(figsize=(6, 4))

plt.plot(x_1, loss_record['train'], c='tab:red', label='train')

plt.plot(x_2, loss_record['dev'], c='tab:cyan', label='dev')

plt.ylim(0.0, 5.)

plt.xlabel('Training steps')

plt.ylabel('MSE loss')

plt.title('Learning curve of {}'.format(title))

plt.legend()

plt.show()

  
  

def plot_pred(dv_set, model, device, lim=35., preds=None, targets=None):

''' Plot prediction of your DNN '''

if preds is None or targets is None:

model.eval()

preds, targets = [], []

for x, y in dv_set:

x, y = x.to(device), y.to(device)

with torch.no_grad():

pred = model(x)

preds.append(pred.detach().cpu())

targets.append(y.detach().cpu())

preds = torch.cat(preds, dim=0).numpy()

targets = torch.cat(targets, dim=0).numpy()

  

figure(figsize=(5, 5))

plt.scatter(targets, preds, c='r', alpha=0.5)

plt.plot([-0.2, lim], [-0.2, lim], c='b')

plt.xlim(-0.2, lim)

plt.ylim(-0.2, lim)

plt.xlabel('ground truth value')

plt.ylabel('predicted value')

plt.title('Ground Truth v.s. Prediction')

plt.show()
```

## Class
class的组成部分有两个：
1. 关于用到的内部函数的定义 def __init__
2. 

```python
class COVID19Dataset(Dataset):
    ''' Dataset for loading and preprocessing the COVID19 dataset '''
    def __init__(self,
                 path,
                 mode='train',
                 target_only=False):
        self.mode = mode

        # Read data into numpy arrays
        with open(path, 'r') as fp:
            data = list(csv.reader(fp))
            data = np.array(data[1:])[:, 1:].astype(float)
        
        if not target_only:
            feats = list(range(93))
        else:
            # TODO: Using 40 states & 2 tested_positive features (indices = 57 & 75)
            pass

        if mode == 'test':
            # Testing data
            # data: 893 x 93 (40 states + day 1 (18) + day 2 (18) + day 3 (17))
            data = data[:, feats]
            self.data = torch.FloatTensor(data)
        else:
            # Training data (train/dev sets)
            # data: 2700 x 94 (40 states + day 1 (18) + day 2 (18) + day 3 (18))
            target = data[:, -1]
            data = data[:, feats]
            
            # Splitting training data into train & dev sets
            if mode == 'train':
                indices = [i for i in range(len(data)) if i % 10 != 0]
            elif mode == 'dev':
                indices = [i for i in range(len(data)) if i % 10 == 0]
            
            # Convert data into PyTorch tensors
            self.data = torch.FloatTensor(data[indices])
            self.target = torch.FloatTensor(target[indices])

        # Normalize features (you may remove this part to see what will happen)
        self.data[:, 40:] = \
            (self.data[:, 40:] - self.data[:, 40:].mean(dim=0, keepdim=True)) \
            / self.data[:, 40:].std(dim=0, keepdim=True)

        self.dim = self.data.shape[1]

        print('Finished reading the {} set of COVID19 Dataset ({} samples found, each dim = {})'
              .format(mode, len(self.data), self.dim))

    def __getitem__(self, index):
        # Returns one sample at a time
        if self.mode in ['train', 'dev']:
            # For training
            return self.data[index], self.target[index]
        else:
            # For testing (no target)
            return self.data[index]

    def __len__(self):
        # Returns the size of the dataset
        return len(self.data)
```

