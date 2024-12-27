# issue : 회원탈퇴 시 댓글 수 변경안됨.

백엔드에서 실수가 있었음. 

SQL의 IN 연산자는 조건에 해당하는 레코드를 고유하게 처리하기 때문에 동일한 postId가 postIds 배열에 여러 개 포함되어 있어도 해당 postId를 딱 한 번만 조건으로 사용하는 문제가 있었음.

기존 코드: 

    // writerId가 특정 user_id인 모든 comment의 postId를 가져옴
    query = `SELECT postId FROM comments WHERE writerId = ?`;
    const [comments] = await req.db.query(query, [user_id]);

    if (comments.length > 0) {
        const postIds = comments.map(comment => comment.postId);

        // 해당 postId들의 comment 값을 한 번에 감소
        query = `UPDATE posts SET comment = comment - 1 WHERE id IN (?)`;
        await req.db.query(query, [postIds]);
    }

<br>
각 postId별로 댓글 개수를 구한 후, 그만큼 줄여줘 해결.
<br>
수정된 코드:

        // writerId가 특정 user_id인 모든 comment의 postId와 그 개수를 그룹화하여 가져옴
        query = `SELECT postId, COUNT(*) as count FROM comments WHERE writerId = ? GROUP BY postId`;
        const [comments] = await req.db.query(query, [user_id]);

        if (comments.length > 0) {
            for (const { postId, count } of comments) {
                // 각 postId에 대해 comment 값을 count만큼 감소
                query = `UPDATE posts SET comment = comment - ? WHERE id = ?`;
                await req.db.query(query, [count, postId]);
            }
        }