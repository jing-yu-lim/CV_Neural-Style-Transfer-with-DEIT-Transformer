# Neural Style Transfer with DEIT Transformer

This is an alternative Pytorch Neural Style Transfer(NST) implementation, adapted from:
https://www.pluralsight.com/guides/artistic-neural-style-transfer-with-pytorch

Instead of using VGG net's layer activations of the content, style and generated images to calculate loss, the encodings from the DEIT transformer encoders are used

## Content Image Reconstruction
The content image reconstruction from noise using DEIT works quite well. However, there are some artefacts patches in the image. This is likely related to the Self-Attention mechanism as the artefacts are congreated in what appears to be 16x16 sized patches, i.e. the size of the input image patches into the transformer. It is interesting to note that the patch artefacts only appears in the later training steps, as the loss values decreases. Many attempts were made to reduce the artefacts by retrieving encodings from different blocks of transformer encoder to calculate the loss. However, it only introduced more artefacts (refer to results below).  

It is also interesting that this works much better than the overall NST (below), and does not suffer from the same problem of the image becoming very low resolution and patch-ey. While this phenomenon occurs at the start of reconstructing the image, the patch-ey effects quickly wanes.

Content Image Reconstruction is available in the second part of the colab notebook in the main code.
### Results using single transformer block encodings:
![content_gen_overall2](https://user-images.githubusercontent.com/79006977/129229839-ce48c412-ecdf-4177-9a23-67a1bc55f2ec.jpg)


https://user-images.githubusercontent.com/79006977/129231289-a82c13f4-e2db-4825-b048-baf6550b35e0.mp4

### Results using ALL transformer block encodings:

![content_gen_overall](https://user-images.githubusercontent.com/79006977/129294982-cc528cdb-3422-4f10-8e78-b23974f2de8e.jpg)

https://user-images.githubusercontent.com/79006977/129294941-a9786a08-d035-40b6-8102-fa089a02f536.mp4


## Full Neural Style Transfer with DEIT transformer
Unfortunately, the results are not as pleasant as the classic NST. This is likely due to the 16x16 image patches used as the inputs to the Vision Transformer. Perhaps the gradient backpropped for each image patch is the same for all 16x16 pixels, rendering the image to be patch-ey. Moreover there is the 'Self-Attention' artefacts as shown above, further causing distortion to the generated image. 

Nonetheless, it is interesting to note that there is some form of style being transferred, indicating that the gram matrices of the transformer encodings represent the style of the images.

![NST_overall2](https://user-images.githubusercontent.com/79006977/129229584-6062444d-85f5-4c4d-8f8e-06296097b8e6.jpg)


https://user-images.githubusercontent.com/79006977/129231323-bd6552ec-f595-425b-af08-42e664549038.mp4



### Comparison with Classic NST
![doggo_classic_overall2](https://user-images.githubusercontent.com/79006977/129231189-715976a3-4b85-4b0e-87c8-1db1625aaa00.jpg)


