<template>
    <div class="footer">
        <img src="@/assets/image/footer1.png">
        NAIVEBBS
        <img src="@/assets/image/footer2.png">
    </div>
</template>

<style scoped>
.footer{
    position: sticky;
    width:100%;
    height:80px;
    background-color: #f5f5f5;
    box-shadow: 5px 0px 10px rgba(0, 0 ,0 ,0.1);
    text-align: center;
    line-height: 80px;
    margin-top:50px;
    font-size: 26px;
    font-family: 'Times New Roman', Times, serif;
}
img{
    height: 60px;
    vertical-align: middle;
    margin: 0 10px;
}
@media screen and (max-width: 768px) {
    .footer{
        font-size:20px;
    }
}
</style>