# Neural Style Transfer with DEIT Transformer

This is an alternative Pytorch Neural Style Transfer(NST) implementation, adapted from this [link](https://www.pluralsight.com/guides/artistic-neural-style-transfer-with-pytorch), and implemented using Facebook's Data Efficient Image Transformer (DEIT). 

Instead of using VGG net's layer activations of the content, style and generated images to calculate loss, the encodings from the DEIT transformer encoders are used

## Content Image Reconstruction
The content image reconstruction from noise using DEIT works quite well. However, there are some artefacts patches in resultant images. This is likely related to the Self-Attention mechanism as the artefacts are congregated in what appears to be 16x16 sized patches, i.e. the size of the input image patches into the transformer. It is interesting to note that the patch artefacts only appears in the later training steps, as the loss values decreases. Many attempts were made to reduce the artefacts by retrieving encodings from different blocks of the transformer encoder to calculate the loss. However, it only introduced more artefacts (refer to results below).  

It is also interesting that this works much better than the overall NST (below), and does not suffer from the same problem of the image becoming very low resolution and patch-ey. While this phenomenon occurs at the start of reconstructing the image, the patch-ey effects quickly wanes.

### Results using single transformer block encodings:
<p align="center">
<img width="725" height="353" src="https://user-images.githubusercontent.com/79006977/129229839-ce48c412-ecdf-4177-9a23-67a1bc55f2ec.jpg">
</p>

https://user-images.githubusercontent.com/79006977/129231289-a82c13f4-e2db-4825-b048-baf6550b35e0.mp4

### Results using ALL transformer block encodings:

<p align="center">
<img width="725" height="353" src="https://user-images.githubusercontent.com/79006977/129294982-cc528cdb-3422-4f10-8e78-b23974f2de8e.jpg">
</p>

https://user-images.githubusercontent.com/79006977/129294941-a9786a08-d035-40b6-8102-fa089a02f536.mp4


## Full Neural Style Transfer with DEIT transformer
Unfortunately, the results are not as pleasant as the classic NST. This is likely due to the 16x16 image patches used as the inputs to the transformer encoder. One possible explanation is that the gradient backpropped for each image patch is the same for all 16x16 pixels, resulting in the patches adopting similar values. Moreover there is the 'Self-Attention' artefact as shown above, further causing distortion to the generated image. 

Nonetheless, it is interesting to note that there is some form of style being transferred, indicating potential representation of image style using the transformer encodings' gram matrices implemented here (explained below).

![NST_overall2](https://user-images.githubusercontent.com/79006977/129229584-6062444d-85f5-4c4d-8f8e-06296097b8e6.jpg)


https://user-images.githubusercontent.com/79006977/129231323-bd6552ec-f595-425b-af08-42e664549038.mp4



### Comparison with Classic NST
![doggo_classic_overall2](https://user-images.githubusercontent.com/79006977/129231189-715976a3-4b85-4b0e-87c8-1db1625aaa00.jpg)

## Gram Matrix for Transformers
![image](https://user-images.githubusercontent.com/79006977/129343837-f02ac83c-94d7-4a15-b73d-ec55fa9ad59f.png)
(https://towardsdatascience.com/introduction-to-neural-style-transfer-with-tensorflow-99915a5d624a)

In classic NST, the Gram Matrix of the feature maps for the images represents the style of the image. It is implemented by matrix multiplation of a feature map with its height and width flattened (shape of (n_C, n_H x n_W)), and its transpose. 

The intuition of retrieving the gram matrix from the Vision Transformer encodings was based on how the centre-cropped 224x224 input images were tokenized into 16x16 image patches (which are flattened), with an embedding dimension of 768.

A 2D Conv net with a Kernel size of 16, Stride of 16, input channel of 3 and output channel of 768 was used in Vision Transformer for the aforementioned tokenization. Height and width dimensions are then flattened. The result is a set of 'feature maps' with the shape (1 , 196, 768). A class token is concatenated with this 'feature map', and the final shape of the transformer input tokens is (1, 197, 768). Subsequent transformer blocks' encodings share the same shape as the Multi-Head Attention mechanism can be intuited to be making the tokens more expressive.

Hence, the 768 embedding dimension of the encodings can be interpreted as the number of 'channels' of the 'feature maps', and the 197 dimension is simply the flattened  n_H x n_W dimensions plus a positional encoding. The gram matrix was thus obtained by manipulating the encodings to a shape of (embed_dim, H * W) i.e. (768, 197), and matrix mutlplying it with its transpose.
