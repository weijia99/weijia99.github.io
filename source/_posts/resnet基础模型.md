---
title: resnet基础模型
date: 2023-02-17 17:09:49
tags: [深度学习]
passw: weijia

---

# 1.数据集搭建

使用经典的csv读取数据，然后再进行构建自己的dataset的时候，通过csv ，直接获得图片地址，使用pil来读取图像，使用int强转label（label本来是是string，通过构建字典dict来得到的）

```python
def __getitem__(self, idx):
    img, label = self.images[idx], self.labels[idx]
    # 返回图像,下面的std是来自imagenet
    tf = transforms.Compose([
        lambda x: Image.open(x).convert('RGB'),  # string path= > image data
        transforms.Resize((int(self.resize * 1.25), int(self.resize * 1.25))),
        transforms.RandomRotation(15),
        transforms.CenterCrop(self.resize),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406],
                             std=[0.229, 0.224, 0.225])
    ])

    img = tf(img)
    label = torch.tensor(label)

    return img, label
```



直接读取所有的文件，构建csv

```python
    def make_csv(self):
        """
        构建csv，返回当前路径的图片的属性
        :return:
        """
        file_name = './label.csv'
        type_dir = glob.glob(os.path.join(self.root, '*'))
        with open(file_name, 'w') as csvfile:
            writer = csv.writer(csvfile)
            for type in type_dir:
                person_dir = glob.glob(os.path.join(type, '*'))
                for person in person_dir:
                    emotion_dir = glob.glob(os.path.join(person, '*'))
                    for emotion in emotion_dir:
                        label = emotion.split('/')[-1]
                        img_dir = glob.glob(os.path.join(emotion, '*.jpeg'))
                        for img in img_dir:
                            writer.writerow([img, label])
                    print("writting one person")

        print("process ok")

```

、

之后就是进行分割数据集，train，val，test=6:2：2

```python
    def __init__(self, path, resize, mode):
        super(NIREmotionDatasets, self).__init__()
        # 进行自定义标签
        self.class_dir = {"Anger": 0, "Disgust": 1, "Fear": 2, "Happiness": 3, "Sadness": 4, "Surprise": 5}
        self.root = path
        self.resize = resize
        self.images, self.labels = self.load_csv('./label.csv')
        if mode == "train":
            self.images = self.images[:int(0.6 * len(self.images))]
            self.labels = self.labels[:int(0.6 * len(self.labels))]
        elif mode == 'val':  # 20% = 60%->80%
            self.images = self.images[int(0.6 * len(self.images)):int(0.8 * len(self.images))]
            self.labels = self.labels[int(0.6 * len(self.labels)):int(0.8 * len(self.labels))]
        elif mode == 'test':  # 20% = 80%->9999%
            self.images = self.images[int(0.8 * len(self.images)):int(0.99999 * len(self.images))]
            self.labels = self.labels[int(0.8 * len(self.labels)):int(0.99999 * len(self.labels))]

```



# 2.train模板

因为resnet在timm库里直接由，我们直接进行调用

构建代码步骤

1. 构建argparse，直接复制粘贴
2. 构建dataloader，传入我们的datasets，然后进行shuttle，设置batch
3.  进行设置crition，adam还有model，模型直接进行调用，然后传入之前的pth，使用load
4. 设置model的最后一层为class层数 ，进行修改后放入device
5. 之后就是for语句了，在epoch里面
6. 首先是进行验证，验证，是计算正确率（这个就是，accurate的计算方法
7. 之后就是train，train需要设置为train模式，然后这个是计算loss的
8. 上面的常规就是zerograd，然后计算pred，pred与y计算loss，之后，loss进行反串，优化器进行更新
9. 一轮后更新学习率lr



```python
best_acc = 0
    print(args.input_size)
    datasets_train = build_datasets('train', args)
    datasets_val = build_datasets('test', args)
    emotion_idx = datasets_train.class_dir
    train_loader = torch.utils.data.DataLoader(datasets_train, batch_size=args.batch_size, shuffle=True,
                                               num_workers=args.num_workers)
    val_loader = torch.utils.data.DataLoader(datasets_val, batch_size=args.batch_size, shuffle=True,
                                             num_workers=args.num_workers)
    model_resnet = resnet18(pretrained=True)
    checkpoint = torch.load('/home/tonnn/.nas/weijia/work/fer/baseline/resnet_base/resnet18_msceleb .pth')
    model_resnet.load_state_dict(checkpoint['state_dict'], strict=True)

    model = nn.Sequential(*list(model_resnet.children())[:-1],
                          Flatten(),
                          nn.Linear(512, 6)
                          ).to(device)
    val_num = len(val_loader)
    train_num = len(train_loader)
    print("using {} images for training, {} images for validation.".format(train_num, val_num))

    params = list(model.parameters())
    optimizer = torch.optim.SGD(params, lr=args.lr, weight_decay=1e-4, momentum=0.9)
    scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.1)

    criteon = nn.CrossEntropyLoss()

    # 训练start
    for epoch in range(args.epochs):
        print(f'Epoch {epoch}')
        print(f'length of dataloader_train is {len(train_loader)}')

        acc = 0.0
        if epoch % 1 == 0:
            print("Evaluating...")

            model.eval()

            with torch.no_grad():
                for x, y in val_loader:
                    x, y = x.to(device), y.to(device)
                    logits = model(x)
                    pred = logits.argmax(dim=1)
                    # 找出最大值作为答案
                    acc += torch.eq(pred, y.to(device)).sum().item()

            val_acc = acc / len(val_loader)
            print('[epoch %d]  val_accuracy: %.3f' %
                  (epoch + 1,  val_acc))

            if val_acc > best_acc:
                best_acc = val_acc
                torch.save(model.state_dict(), 'accuracy_loss_change_train_50.mdl')

        model.train()
        for step,(x,y) in enumerate(train_loader):
            x, y = x.to(device), y.to(device)
            logits = model(x)
            loss = criteon(logits,y)

            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

        scheduler.step()
        # 更新 lr
```





# 3.进行预测test还有读取pth继续进行训练



预测函数predict，我们新建一个文件，然后还是要把预测的图片路径传入进来，然后就是进行加载模型，载入权重，之后使用nograd来进行计算，最后还需要id到属性的字典来进行映射

1. 使用dataset读取图片传入到dataloader
2. 设置enviroment来进行多卡推理
3. 设置模型，进行加载权重，使用模型的load'函数进行加载
4. 最后就是with nograd'，进行推理，推理的时候，数据也要传到cuda上面
5. 然后找到最大的索引，之后通过反向查询，得到他的名称

```python
 os.environ['CUDA_VISIBLE_DEVICES'] = '0,1'

    device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
    test_datasets = build_datasets("test", get_args_parser().parse_args())

    test_loader = torch.utils.data.DataLoader(
        test_datasets, batch_size=args.batch_size, shuffle=False,
        num_workers=args.num_workers)

    class_idx = test_datasets.class_dir

    # 加载模型
    pth_path = 'accuracy_loss_change_train_50.mdl'
    model = resnet18()
    model = nn.Sequential(*list(model.children())[:-1],
                          Flatten(),
                          nn.Linear(512, 6)
                          ).to(device)
    model.load_state_dict(torch.load(pth_path))
    print('loaded from ckpt!')
    model = nn.DataParallel(model)
    model = model.cuda()
    model.eval()
    acc = 0
    ans = []
    with torch.no_grad():
        for x, y in test_loader:
            x, y = x.cuda(), y.cuda()
            logits = model(x)
            pred = logits.argmax(dim=1)

            for x1 in pred:
                ans.append(find(class_idx, int(x1)))
            # 找出最大值作为答案
            acc += torch.eq(pred, y.to(device)).sum().float().item()

```



继续训练和之前的predict一样，看args的startt是不是0，不是0，就进行加载权重

```
  if args.start_epoch !=0:
        model.load_state_dict = torch.load('accuracy_loss_change_train_1003.mdl')
  
```



# 4.pytorch多卡运行，读取还有保存权重

多卡运行，上面已经有操作了

1. 首先设置os的黄精为0,1
2. 然后加载到dataparallel上面
3. 之后模型移动到cuda
4. 然后在训练和预测的时候数据移动到cuda



和上面没有什么差别，支部够要移动一下，普通是移动到device

```python
 criteon = nn.CrossEntropyLoss()
    # if args.start_epoch !=0:
    #     model.load_state_dict = torch.load('accuracy_loss_change_train_1003.mdl')
    #     # model.load_state_dict(torch.load('accuracy_loss_change_train_3.mdl'))

    model = nn.DataParallel(model)
    model = model.cuda()
    # 训练start
    for epoch in range(0,args.epochs):
        print(f'Epoch {epoch}')
        print(f'length of dataloader_train is {len(train_loader.dataset)}')

        acc = 0.0
        if epoch % 1 == 0:
            print("Evaluating...")

            model.eval()

            with torch.no_grad():
                for x, y in val_loader:
                    x, y = x.cuda(), y.cuda()
                    logits = model(x)
                    pred = logits.argmax(dim=1)
                    # 找出最大值作为答案
                    acc += torch.eq(pred, y.cuda()).sum().item()

            val_acc = acc / len(val_loader.dataset)
            print('[epoch %d]  val_accuracy: %.3f' %
                  (epoch + 1,  val_acc))

            if val_acc > best_acc:
                best_acc = val_acc
                file_name = '/home/tonnn/.nas/weijia/work/fer/baseline/resnet_base/checkpoint/accuracy_loss_change_train_'+str(epoch)+'.pth'
                torch.save(model.state_dict(), file_name)

        model.train()
        for step,(x,y) in enumerate(train_loader):
            x, y = x.cuda(), y.cuda()
            logits = model(x)
            loss = criteon(logits,y)

            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            print(r"train epoch[{}/{}] loss:{:.3f}".format(epoch + 1,
                                                                     args.epochs,
                                                                     loss.item()))

        scheduler.step()
        # 更新 lr
```





# 附录：mtcnn进行裁剪

