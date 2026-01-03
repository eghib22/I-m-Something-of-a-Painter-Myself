# I-m-Something-of-a-Painter-Myself
ამ პროექტის მიზანია unpaired image-to-image translation ამოცანის გადაწყვეტა, რეალური ფოტოების გარდაქმნა Claude Monet-ს სტილში შესრულებულ ნახატებად. პროექტი ეფუძნება CycleGAN architecture-ს, რომელიც საშუალებას იძლევა ორ დომეინს შორის ტრანსფორმაციას paired data-ს გარეშე.


გამოყენებულია ორი დამოუკიდებელი dataset:
  photo_jpg — რეალური ფოტოები
  monet_jpg — Claude Monet-ის ნახატები

ეს მონაცემები არ არის წყვილებად დაჯგუფებული, ანუ კონკრეტულ ფოტოს არ შეესაბამება კონკრეტული ნახატი. სწორედ ამიტომ გამოვიყენე CycleGAN არქიტექტურა. 

დავწერე MonetPhotoDataset(Dataset) კლასი, რომელიც:
  თითოეულ __getitem__ გამოძახებაზე აბრუნებს ერთ Monet-ს სურათს, ერთ Photo სურათს (random არჩევით)
__len__ განისაზღვრება Monet სურათების რაოდენობით

გამოყენებულია DataLoader შემდეგი პარამეტრებით:
  batch_size = 1 
  shuffle = True
  num_workers = 2

გამოყენებულია შემდეგი transform pipeline:
  Resize(286)
  RandomCrop(256)
  RandomHorizontalFlip()
  ToTensor()
  Normalize(mean=0.5, std=0.5)

ეს ნაბიჯები საჭიროა data augmentation-ისთვის, სურათების ერთ ზომაზე დასაყვანად და მონაცემების [-1, 1] დიაპაზონში გადასაყვანად (Tanh activation-ისთვის)
ორიგინალ CycleGAN paper-ში გამოყენებულია ResNet-based generator ამიტომ მეც ეგ გამოვიყენე.
ResnetBlock წარმოადგენს residual block-ს:
  ორი convolution layer
  InstanceNorm
  ReLU activation
  skip connection (x + F(x))
ეს საშუალებას იძლევა:
  შენარჩუნდეს low-level ინფორმაცია
  გაუმჯობესდეს gradient flow
  თავიდან ავიცილოთ vanishing gradient
Generator შედგება შემდეგი ნაწილებისგან:
  Initial convolution (7×7)
  Downsampling (2 convolution layer, stride=2)
  Residual blocks (9–10 block)
  Upsampling (ConvTranspose2d)
  Final Tanh activation

Loss Functions
CycleGAN-ის training იყენებს რამდენიმე loss-ს:
Adversarial Loss - აიძულებს Generator-ს შექმნას რეალისტური გამოსახულებები.
Cycle Consistency Loss (lambda_cycle)
უზრუნველყოფს, რომ Photo → Monet → Photo ≈ original Photo და Monet → Photo → Monet ≈ original Monet
Identity Loss (lambda_identity)
გამოიყენება იმისთვის, რომ Monet → Generator(Monet) ≈ Monet
ფერები და brightness არ შეიცვალოს ზედმეტად
